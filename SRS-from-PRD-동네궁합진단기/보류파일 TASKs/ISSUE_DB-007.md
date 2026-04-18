---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[DB] DB-007: NextAuth.js Prisma Adapter 스키마 (Account, Session, VerificationToken) 통합"
labels: 'database, auth, backend, priority:high'
assignees: ''
---

## :dart: Summary
- **기능명:** [DB-007] NextAuth.js Prisma Adapter 스키마 (Account, Session, VerificationToken) 통합
- **목적:** SRS REQ-FUNC-029에 따라 **NextAuth.js v5 (Auth.js)** 기반 카카오·네이버 소셜 로그인을 지원하기 위해, NextAuth.js Prisma Adapter가 요구하는 표준 스키마(Account, Session, VerificationToken)를 DB-002에서 정의한 User 모델과 통합한다. 이 태스크가 완료되면 NextAuth.js가 Prisma를 통해 사용자 UPSERT, OAuth 계정 연결, 세션 관리를 자동으로 처리할 수 있다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 인증 요구사항: [`/TASKS/SRS_v1.md#4.1.6`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — REQ-FUNC-029 (NextAuth.js v5 세션 전략)
- SRS 인증 시퀀스: [`/TASKS/SRS_v1.md#6.3.6`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — NextAuth → Prisma: User UPSERT + Account 저장 (Prisma Adapter)
- SRS 세션 보안: [`/TASKS/SRS_v1.md#4.2.3`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — REQ-NF-018 (httpOnly cookie, maxAge 7일, updateAge 15분)
- SRS API: [`/TASKS/SRS_v1.md#6.1 API-07/08`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — NextAuth Route Handler
- SRS 컴포넌트: [`/TASKS/SRS_v1.md#6.6`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — Auth Module (NextAuth.js v5) 
- NextAuth Prisma Adapter 공식 문서: https://authjs.dev/getting-started/adapters/prisma

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] **T1.** NextAuth 의존성 설치: `npm install next-auth@beta @auth/prisma-adapter`
- [ ] **T2.** DB-002에서 정의한 User 모델을 NextAuth Adapter 호환으로 확장:
  ```prisma
  model User {
    id            String       @id @default(uuid())
    email         String       @unique
    emailVerified DateTime?    @map("email_verified")
    name          String?
    image         String?
    authProvider  AuthProvider? @map("auth_provider")
    mode          ServiceMode  @default(couple)
    createdAt     DateTime     @default(now()) @map("created_at")
    updatedAt     DateTime     @updatedAt @map("updated_at")

    // NextAuth Relations
    accounts      Account[]
    sessions      Session[]

    // Business Relations
    diagnoses     Diagnosis[]
    payments      Payment[]

    @@map("users")
  }
  ```
  > ⚠️ `name`, `image`, `emailVerified` 필드 추가 — NextAuth Adapter 필수. `authProvider`를 nullable로 변경 (NextAuth가 provider 정보를 Account 테이블에 별도 저장하므로).
- [ ] **T3.** Account 모델 정의 (NextAuth Adapter 표준):
  ```prisma
  model Account {
    id                String  @id @default(uuid())
    userId            String  @map("user_id")
    type              String
    provider          String
    providerAccountId String  @map("provider_account_id")
    refresh_token     String? @db.Text
    access_token      String? @db.Text
    expires_at        Int?
    token_type        String?
    scope             String?
    id_token          String? @db.Text
    session_state     String?

    user User @relation(fields: [userId], references: [id], onDelete: Cascade)

    @@unique([provider, providerAccountId])
    @@map("accounts")
  }
  ```
- [ ] **T4.** Session 모델 정의 (NextAuth Adapter 표준):
  ```prisma
  model Session {
    id           String   @id @default(uuid())
    sessionToken String   @unique @map("session_token")
    userId       String   @map("user_id")
    expires      DateTime

    user User @relation(fields: [userId], references: [id], onDelete: Cascade)

    @@map("sessions")
  }
  ```
- [ ] **T5.** VerificationToken 모델 정의 (NextAuth Adapter 표준):
  ```prisma
  model VerificationToken {
    identifier String
    token      String   @unique
    expires    DateTime

    @@unique([identifier, token])
    @@map("verification_tokens")
  }
  ```
- [ ] **T6.** 마이그레이션 실행: `npx prisma migrate dev --name add-nextauth-adapter-tables`
- [ ] **T7.** NextAuth 설정 파일 기초 생성 (`src/lib/auth.ts`):
  ```typescript
  import { PrismaAdapter } from "@auth/prisma-adapter"
  import NextAuth from "next-auth"
  import { prisma } from "@/lib/db"

  export const { handlers, auth, signIn, signOut } = NextAuth({
    adapter: PrismaAdapter(prisma),
    // Provider 설정은 CMD-AUTH-001에서 추가
    providers: [],
    session: {
      strategy: "database",
      maxAge: 7 * 24 * 60 * 60,      // 7일 (REQ-NF-018)
      updateAge: 15 * 60,              // 15분 (REQ-NF-018)
    },
  })
  ```
- [ ] **T8.** NextAuth Route Handler 생성 (`src/app/api/auth/[...nextauth]/route.ts`):
  ```typescript
  import { handlers } from "@/lib/auth"
  export const { GET, POST } = handlers
  ```
- [ ] **T9.** Prisma Studio에서 Account·Session·VerificationToken 테이블 생성 확인

## :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: NextAuth Prisma Adapter 연결 확인**
- **Given:** Prisma 마이그레이션 완료 + NextAuth 설정 파일 생성
- **When:** `/api/auth/providers` 엔드포인트에 GET 요청
- **Then:** NextAuth가 정상 응답을 반환 (빈 providers 목록이라도 200 OK). DB 연결 에러 없음.

**Scenario 2: User 모델 NextAuth 호환성**
- **Given:** accounts, sessions 테이블이 존재
- **When:** PrismaAdapter를 통해 `createUser({ email, name, image })`가 호출됨 (NextAuth 내부 동작)
- **Then:** users 테이블에 User 레코드가 생성되고, `mode` 기본값 'couple', `createdAt` 자동 설정.

**Scenario 3: Account-User CASCADE 삭제**
- **Given:** User에 연결된 Account 레코드 존재
- **When:** User를 삭제
- **Then:** Account 레코드도 CASCADE 삭제.

**Scenario 4: provider+providerAccountId UNIQUE 제약**
- **Given:** `provider="kakao"`, `providerAccountId="12345"`인 Account 존재
- **When:** 동일 provider+providerAccountId로 Account 생성 시도
- **Then:** Prisma P2002 에러 발생. (동일 OAuth 계정 중복 연결 차단)

**Scenario 5: Session 모델 정합성**
- **Given:** User 레코드 존재
- **When:** `prisma.session.create({ data: { sessionToken: "token_abc", userId, expires: futureDate } })`
- **Then:** sessions 테이블에 레코드 생성. `sessionToken`은 UNIQUE.

## :gear: Technical & Non-Functional Constraints
- **NextAuth v5 (Auth.js) 호환:** NextAuth v5 Prisma Adapter의 공식 스키마를 정확히 준수. 필드명·타입 변경 금지.
- **User 모델 확장:** SRS §6.2.1 원본 User 필드 + NextAuth 필수 필드(`name`, `image`, `emailVerified`)를 병합. `authProvider`는 Account 테이블에서 관리하므로 nullable로 변경.
- **세션 전략 (REQ-NF-018):** `strategy: "database"` — httpOnly cookie에 세션 토큰만 저장, 실제 세션 데이터는 DB에 저장. JWT 전략 사용 금지.
- **토큰 보안:** `access_token`, `refresh_token`, `id_token`은 `@db.Text`로 저장 — 길이 제한 없이 전체 토큰 보존.
- **SQLite 호환성 주의:** SQLite에서는 `@db.Text` 어노테이션이 무시됨 (TEXT가 기본). PostgreSQL에서만 유효.

## :checkered_flag: Definition of Done (DoD)
- [ ] User 모델에 NextAuth 필수 필드(`name`, `image`, `emailVerified`) 추가됨
- [ ] Account, Session, VerificationToken 모델이 NextAuth Adapter 표준과 일치
- [ ] `npx prisma migrate dev` 성공
- [ ] `src/lib/auth.ts`에 PrismaAdapter 기본 설정 완료
- [ ] `src/app/api/auth/[...nextauth]/route.ts` Route Handler 생성
- [ ] `/api/auth/providers` 접근 시 200 OK 응답 확인
- [ ] Account `@@unique([provider, providerAccountId])` 복합 유니크 제약 확인

## :construction: Dependencies & Blockers
- **Depends on:** DB-002 (USER 테이블 — User 모델 확장)
- **Blocks:**
  - API-001 (Auth 도메인 DTO — 세션 객체·콜백 타입)
  - CMD-AUTH-001 (카카오 OAuth Provider 설정)
  - CMD-AUTH-002 (네이버 OAuth Provider 설정)
  - CMD-AUTH-003 (세션 전략 구현)
  - MOCK-005 (OAuth Mock 데이터)
