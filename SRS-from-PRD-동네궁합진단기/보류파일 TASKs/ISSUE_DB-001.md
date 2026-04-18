---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[DB] DB-001: Prisma 프로젝트 초기화 및 datasource 설정 (SQLite/Supabase 전환 가능 구조)"
labels: 'database, backend, priority:critical'
assignees: ''
---

## :dart: Summary
- **기능명:** [DB-001] Prisma 프로젝트 초기화 및 datasource 설정 (SQLite/Supabase 전환 가능 구조)
- **목적:** SRS CON-11(C-TEC-003)에 따라 **Prisma ORM**을 프로젝트에 통합하고, 로컬 개발 시 **SQLite**, 프로덕션 배포 시 **Supabase PostgreSQL**을 환경변수(`DATABASE_URL`) 하나로 전환할 수 있는 datasource 구조를 확립한다. 이 태스크는 전체 데이터 레이어(DB-002 ~ DB-010)의 출발점이자, 모든 Server Action·Route Handler가 참조할 DB 클라이언트의 SSOT(Single Source of Truth)이다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS ERD: [`/TASKS/SRS_v1.md#6.2.0 ERD`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — 4개 핵심 엔터티 관계도
- SRS 제약사항: [`/TASKS/SRS_v1.md#1.2.3 Constraints`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — CON-11 (C-TEC-003: Prisma + SQLite/Supabase)
- SRS 확장성: [`/TASKS/SRS_v1.md#4.2.7`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — REQ-NF-041 (지역 확장 시 설정 파일 + 데이터 적재만으로 가능)
- SRS 컴포넌트: [`/TASKS/SRS_v1.md#6.6 Component Diagram`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — Prisma ORM → DevDB(SQLite) / ProdDB(Supabase PostgreSQL)

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] **T1.** Prisma 의존성 설치: `npm install prisma @prisma/client` + devDependency 설정
- [ ] **T2.** `npx prisma init` 실행 → `prisma/schema.prisma` 파일 생성
- [ ] **T3.** `schema.prisma` datasource 설정 — 환경변수 기반 DB 전환:
  ```prisma
  datasource db {
    provider = "postgresql"    // 프로덕션 기본
    url      = env("DATABASE_URL")
  }

  generator client {
    provider = "prisma-client-js"
  }
  ```
  > ⚠️ SQLite 전환 시 `provider = "sqlite"` + `DATABASE_URL="file:./dev.db"` 로 변경.
  > Prisma는 provider 동적 전환을 지원하지 않으므로, 로컬에서는 `prisma/schema.dev.prisma` 별도 관리하거나 `.env.local`에서 PostgreSQL 호환 URL 사용 권장.
- [ ] **T4.** Prisma Client 싱글톤 인스턴스 생성 (`src/lib/db.ts`):
  ```typescript
  import { PrismaClient } from '@prisma/client'

  const globalForPrisma = globalThis as unknown as {
    prisma: PrismaClient | undefined
  }

  export const prisma =
    globalForPrisma.prisma ??
    new PrismaClient({
      log: process.env.NODE_ENV === 'development'
        ? ['query', 'error', 'warn']
        : ['error'],
    })

  if (process.env.NODE_ENV !== 'production') {
    globalForPrisma.prisma = prisma
  }
  ```
- [ ] **T5.** `.env.local`에 로컬 개발용 `DATABASE_URL` 설정 (Supabase 무료 tier pooler URL 또는 SQLite 경로)
- [ ] **T6.** `.gitignore`에 `prisma/dev.db`, `prisma/dev.db-journal`, `.env.local` 추가 확인
- [ ] **T7.** `npx prisma db push` 또는 `npx prisma migrate dev --name init` 로 초기 마이그레이션 실행 (빈 스키마 상태 정상 확인)
- [ ] **T8.** Prisma Studio 동작 확인: `npx prisma studio` → 브라우저에서 DB GUI 접근 가능

## :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: Prisma Client 초기화 성공**
- **Given:** `DATABASE_URL` 환경변수가 유효한 값으로 설정됨
- **When:** `src/lib/db.ts`에서 `prisma` 인스턴스를 import하여 `prisma.$connect()`를 호출함
- **Then:** DB 연결이 성공하고, 에러 없이 `PrismaClient` 인스턴스가 반환된다.

**Scenario 2: 환경변수 누락 시 명확한 에러**
- **Given:** `.env.local` 파일에서 `DATABASE_URL`이 제거됨
- **When:** `npm run dev`로 개발 서버를 시작함
- **Then:** Prisma가 `DATABASE_URL is not set` 에러를 명확하게 출력하며, 서버가 크래시 없이 에러 메시지를 표시한다.

**Scenario 3: 마이그레이션 성공**
- **Given:** `schema.prisma`에 datasource만 정의된 빈 스키마 상태
- **When:** `npx prisma migrate dev --name init`을 실행함
- **Then:** 마이그레이션이 성공하고, `prisma/migrations/` 디렉토리에 초기 마이그레이션 폴더가 생성된다.

**Scenario 4: Prisma Studio 접근**
- **Given:** 마이그레이션이 완료된 상태
- **When:** `npx prisma studio`를 실행함
- **Then:** 브라우저에서 Prisma Studio GUI가 열리고, DB 테이블 목록이 표시된다 (현재는 빈 상태).

**Scenario 5: Hot Reload 시 연결 누수 방지**
- **Given:** 개발 서버가 `npm run dev`로 실행 중
- **When:** 소스 코드를 수정하여 Hot Module Replacement가 10회 이상 발생함
- **Then:** `PrismaClient` 인스턴스가 글로벌 싱글톤으로 관리되어, DB 연결 수가 1개를 유지한다 (연결 누수 0건).

## :gear: Technical & Non-Functional Constraints
- **ORM 강제 (C-TEC-003):** Prisma ORM 외 다른 ORM(TypeORM, Drizzle 등) 사용 금지.
- **Provider 호환성:** 프로덕션은 반드시 PostgreSQL. SQLite는 로컬 개발 편의용. 두 provider 간 비호환 기능(JSON 필드 등) 사용 시 주석으로 명시.
- **연결 풀링:** Supabase PostgreSQL 사용 시 Supabase Pooler(PgBouncer) URL 사용 필수 — 서버리스 환경 연결 풀 소진 방지.
- **성능:** `PrismaClient` 인스턴스는 반드시 싱글톤 패턴 적용. Next.js 개발 모드 HMR 시 인스턴스 중복 생성 금지.
- **로깅:** 개발 환경에서만 `query` 로그 활성화. 프로덕션은 `error` 레벨만.

## :checkered_flag: Definition of Done (DoD)
- [ ] `prisma/schema.prisma` 파일이 존재하며, datasource 설정이 `DATABASE_URL` 환경변수를 참조함
- [ ] `src/lib/db.ts`에 싱글톤 PrismaClient가 구현됨
- [ ] `npx prisma migrate dev --name init` 마이그레이션 성공 (빈 스키마)
- [ ] `npx prisma studio` 실행 후 브라우저에서 GUI 접근 확인
- [ ] `.env.local.example`에 `DATABASE_URL` 키가 문서화됨
- [ ] `.gitignore`에 `prisma/dev.db`, `.env.local` 포함 확인

## :construction: Dependencies & Blockers
- **Depends on:** INFRA-001 (Next.js 프로젝트 초기화)
- **Blocks:**
  - DB-002 (USER 테이블 스키마)
  - DB-003 (DIAGNOSIS 테이블 스키마)
  - DB-005 (PAYMENT 테이블 스키마)
  - DB-006 (SavedSearch 테이블 스키마)
  - DB-007 (NextAuth Adapter 스키마)
  - DB-008 (행정동 코드 Seed 데이터)
  - DB-009 (경찰청 범죄 통계 캐시)
  - DB-010 (교육부 학교 배정 구역)
  - *간접적으로 Step 2 전체 Command/Query 태스크를 Block*
