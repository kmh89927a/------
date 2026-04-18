---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[DB] DB-002: USER 테이블 Prisma 스키마 정의 및 마이그레이션"
labels: 'database, backend, priority:critical'
assignees: ''
---

## :dart: Summary
- **기능명:** [DB-002] USER 테이블 Prisma 스키마 정의 및 마이그레이션
- **목적:** SRS §6.2.1에 정의된 USER 엔터티를 Prisma 스키마로 구현한다. USER 테이블은 DIAGNOSIS, PAYMENT 테이블이 FK로 참조하며, NextAuth.js Adapter의 User 모델과도 통합되어야 하는 **핵심 기반 엔터티**이다. OAuth 소셜 로그인(카카오/네이버) 사용자 정보를 안전하게 저장하고, 서비스 모드(couple/single) 관리를 지원한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS USER 데이터 모델: [`/TASKS/SRS_v1.md#6.2.1 USER`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — 필드명, 타입, 제약조건 전체 명세
- SRS ERD: [`/TASKS/SRS_v1.md#6.2.0 ERD`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — USER ↔ DIAGNOSIS, USER ↔ PAYMENT 관계
- SRS 인증 요구사항: [`/TASKS/SRS_v1.md#4.1.6`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — REQ-FUNC-029 (NextAuth.js 카카오/네이버 소셜 로그인)
- SRS 클래스 다이어그램: [`/TASKS/SRS_v1.md#6.7 CLD`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — User 클래스 속성 및 메서드
- SRS 보안: [`/TASKS/SRS_v1.md#4.2.3`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — REQ-NF-017 (개인정보 AES-256 암호화 저장)

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] **T1.** `prisma/schema.prisma`에 User 모델 정의:
  ```prisma
  enum AuthProvider {
    kakao
    naver
  }

  enum ServiceMode {
    couple
    single
  }

  model User {
    id           String       @id @default(uuid())
    email        String       @unique
    authProvider AuthProvider @map("auth_provider")
    mode         ServiceMode  @default(couple)
    createdAt    DateTime     @default(now()) @map("created_at")
    updatedAt    DateTime     @updatedAt @map("updated_at")

    // Relations (후속 태스크에서 추가됨)
    diagnoses    Diagnosis[]
    payments     Payment[]

    @@map("users")
  }
  ```
- [ ] **T2.** Enum 타입 정의 — `AuthProvider` (kakao | naver), `ServiceMode` (couple | single)
- [ ] **T3.** 인덱스 설정:
  - `email` 필드: UNIQUE 인덱스 (스키마 @unique 제약조건)
  - `authProvider` 필드: 조회 성능을 위한 일반 인덱스 검토 (MVP 단계에서는 선택)
- [ ] **T4.** 마이그레이션 실행: `npx prisma migrate dev --name add-user-table`
- [ ] **T5.** 마이그레이션 SQL 파일 검증 — 생성된 SQL이 SRS §6.2.1 스펙과 일치하는지 확인:
  - `id`: UUID PK, NOT NULL
  - `email`: VARCHAR(255), UNIQUE, NOT NULL
  - `auth_provider`: ENUM, NOT NULL
  - `mode`: ENUM, NOT NULL, DEFAULT 'couple'
  - `created_at`: TIMESTAMP, NOT NULL, DEFAULT NOW()
  - `updated_at`: TIMESTAMP, NOT NULL
- [ ] **T6.** TypeScript 타입 생성: `npx prisma generate` → `@prisma/client`에서 `User` 타입 자동 생성 확인
- [ ] **T7.** Prisma Studio에서 User 테이블 확인 + 테스트 레코드 수동 삽입/삭제 검증

## :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: 정상적인 User 레코드 생성**
- **Given:** 마이그레이션이 완료된 빈 users 테이블
- **When:** Prisma Client로 `prisma.user.create({ data: { email: "test@example.com", authProvider: "kakao" } })`를 실행함
- **Then:** users 테이블에 레코드가 생성되고, `id`는 UUID 형식, `mode`는 기본값 'couple', `created_at`과 `updated_at`이 자동 설정된다.

**Scenario 2: 이메일 중복 시 Unique Constraint 에러**
- **Given:** `test@example.com` 이메일의 User 레코드가 이미 존재함
- **When:** 동일한 이메일로 `prisma.user.create()`를 실행함
- **Then:** Prisma가 `P2002` (Unique constraint violation) 에러를 던지며, 레코드가 생성되지 않는다.

**Scenario 3: 필수 필드 누락 시 에러**
- **Given:** 빈 users 테이블
- **When:** `email` 없이 `prisma.user.create({ data: { authProvider: "naver" } })`를 실행함
- **Then:** Prisma가 NULL constraint violation 에러를 던지며, 레코드가 생성되지 않는다.

**Scenario 4: Enum 값 검증**
- **Given:** 빈 users 테이블
- **When:** `authProvider`에 "google" (허용되지 않은 값)을 설정하여 생성 시도함
- **Then:** TypeScript 타입 체크에서 컴파일 에러가 발생하거나, 런타임에 Prisma 검증 에러가 발생한다.

**Scenario 5: updatedAt 자동 갱신**
- **Given:** User 레코드가 존재하며 `updated_at` 값이 `T1` 시점임
- **When:** `prisma.user.update({ where: { ... }, data: { mode: "single" } })`를 실행함
- **Then:** `updated_at` 값이 `T1` 이후 새로운 타임스탬프로 자동 갱신된다.

## :gear: Technical & Non-Functional Constraints
- **Prisma 컨벤션:** 모델명은 PascalCase(`User`), DB 테이블명은 snake_case(`users`). `@@map()` 사용.
- **필드 매핑:** camelCase 프로퍼티 → snake_case 컬럼. `@map()` 사용 필수.
- **UUID 전략:** Prisma `@default(uuid())` 사용. DB 레벨 UUID 생성. crypto.randomUUID() 사용하지 않음.
- **NextAuth 호환성:** 후속 DB-007에서 NextAuth Adapter와 통합 시, User 모델에 `name`, `image`, `emailVerified` 필드 추가가 필요할 수 있음. 현 단계에서는 SRS 명세 기준으로만 정의하되, NextAuth 확장을 명시적으로 주석 처리.
- **개인정보 (REQ-NF-017):** `email` 필드는 현 단계에서 평문 저장. SEC-001 태스크에서 AES-256 암호화 적용 예정 — 이를 주석으로 명시.

## :checkered_flag: Definition of Done (DoD)
- [ ] `prisma/schema.prisma`에 User 모델이 SRS §6.2.1과 1:1 매핑됨
- [ ] `npx prisma migrate dev` 성공, `prisma/migrations/` 에 SQL 파일 생성됨
- [ ] `npx prisma generate` 성공, TypeScript에서 `import { User } from '@prisma/client'` 가능
- [ ] Prisma Studio에서 CRUD 수동 검증 완료
- [ ] `@unique` 제약조건이 email 필드에 적용됨 (중복 삽입 시 P2002 에러 확인)
- [ ] Enum 타입(AuthProvider, ServiceMode)이 정의되어 타입 안전성 확보

## :construction: Dependencies & Blockers
- **Depends on:** DB-001 (Prisma 프로젝트 초기화)
- **Blocks:**
  - DB-003 (DIAGNOSIS 테이블 — `user_id FK → User.id`)
  - DB-005 (PAYMENT 테이블 — `user_id FK → User.id`)
  - DB-006 (SavedSearch 테이블 — `user_id FK → User.id`)
  - DB-007 (NextAuth Adapter 스키마 — User 모델 확장)
  - CMD-AUTH-001 ~ 004 (Auth 도메인 전체)
  - API-001 (Auth DTO)
