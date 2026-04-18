---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[DB] DB-003: DIAGNOSIS 테이블 Prisma 스키마 정의 및 마이그레이션"
labels: 'database, backend, priority:critical'
assignees: ''
---

## :dart: Summary
- **기능명:** [DB-003] DIAGNOSIS 테이블 Prisma 스키마 정의 및 마이그레이션
- **목적:** SRS §6.2.2에 정의된 DIAGNOSIS 엔터티를 Prisma 스키마로 구현한다. DIAGNOSIS 테이블은 시스템의 **핵심 비즈니스 엔터티**로, 두 동선 교차 진단(F1), 데드라인 모드(F3), 싱글 모드(F4)의 결과를 저장한다. SHARE_LINK, PAYMENT 테이블이 FK로 참조하며, 교집합 후보 동네(CandidateArea) 결과를 JSONB `filters` 필드에 구조화하여 저장한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS DIAGNOSIS 데이터 모델: [`/TASKS/SRS_v1.md#6.2.2 DIAGNOSIS`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — 필드명, 타입, 제약조건 전체 명세
- SRS ERD: [`/TASKS/SRS_v1.md#6.2.0 ERD`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — DIAGNOSIS 관계: USER(1:N), SHARE_LINK(1:N), PAYMENT(1:N)
- SRS 교차 진단 시퀀스: [`/TASKS/SRS_v1.md#6.3.1`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — saveDiagnosisResult Server Action → Prisma Diagnosis 저장
- SRS 데드라인 모드: [`/TASKS/SRS_v1.md#6.3.3`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — deadline_mode=true, deadline 필드 사용
- SRS API 명세: [`/TASKS/SRS_v1.md#6.1 API-01`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — createDiagnosis() 요청/응답 스펙
- SRS 클래스 다이어그램: [`/TASKS/SRS_v1.md#6.7`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — Diagnosis 클래스
- SRS 보안: [`/TASKS/SRS_v1.md#4.2.3`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — REQ-NF-017 (직장 주소·이사 기한 AES-256 암호화)

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] **T1.** `prisma/schema.prisma`에 Diagnosis 관련 Enum 정의:
  ```prisma
  enum DiagnosisStatus {
    processing
    completed
    expired
  }
  ```
  > ServiceMode enum은 DB-002에서 이미 정의됨 (couple | single)
- [ ] **T2.** Diagnosis 모델 정의:
  ```prisma
  model Diagnosis {
    id           String          @id @default(uuid())
    userId       String          @map("user_id")
    deadline     DateTime?       @db.Date
    status       DiagnosisStatus @default(processing)
    filters      Json            @db.JsonB
    mode         ServiceMode
    deadlineMode Boolean         @default(false) @map("deadline_mode")
    createdAt    DateTime        @default(now()) @map("created_at")

    // Relations
    user         User            @relation(fields: [userId], references: [id], onDelete: Cascade)
    shareLinks   ShareLink[]
    payments     Payment[]

    // Indexes
    @@index([userId])
    @@index([status])
    @@index([userId, status])
    @@map("diagnoses")
  }
  ```
- [ ] **T3.** `filters` JSONB 필드 구조 TypeScript 타입 정의 (`src/types/diagnosis.ts`):
  ```typescript
  export interface DiagnosisFilters {
    maxCommuteMinutes: number;          // 최대 통근 시간 (분)
    budgetRange: {
      min: number;                      // 최소 예산 (만원)
      max: number;                      // 최대 예산 (만원)
    };
    commuteTimeSlot: string;            // 출근 시간대 (예: "07:30")
    transportMode: 'transit' | 'car' | 'both';
    addressA: {
      address: string;
      lat: number;
      lng: number;
    };
    addressB: {
      address: string;
      lat: number;
      lng: number;
    };
  }
  ```
- [ ] **T4.** FK 관계 설정:
  - `userId` → `User.id` (CASCADE 삭제 — 사용자 삭제 시 진단 데이터도 삭제)
  - User 모델에 `diagnoses Diagnosis[]` relation 추가 확인
- [ ] **T5.** 인덱스 설정:
  - `userId`: 사용자별 진단 목록 조회 성능
  - `status`: 상태별 필터링 (processing · completed · expired)
  - `[userId, status]`: 복합 인덱스 — "특정 사용자의 완료된 진단" 조회 최적화
- [ ] **T6.** 마이그레이션 실행: `npx prisma migrate dev --name add-diagnosis-table`
- [ ] **T7.** 마이그레이션 SQL 검증 — SRS §6.2.2 스펙 대비:
  - `id`: UUID PK, NOT NULL
  - `user_id`: UUID FK → users.id, NOT NULL
  - `deadline`: DATE, NULLABLE
  - `status`: ENUM, NOT NULL, DEFAULT 'processing'
  - `filters`: JSONB, NOT NULL
  - `mode`: ENUM, NOT NULL
  - `deadline_mode`: BOOLEAN, NOT NULL, DEFAULT FALSE
  - `created_at`: TIMESTAMP, NOT NULL, DEFAULT NOW()
- [ ] **T8.** Prisma Studio에서 Diagnosis + User 관계 CRUD 검증

## :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: 정상적인 Diagnosis 레코드 생성 (커플 모드)**
- **Given:** User 레코드(`user_id`)가 존재함
- **When:** `prisma.diagnosis.create({ data: { userId: user_id, mode: "couple", filters: { maxCommuteMinutes: 60, ... } } })`를 실행함
- **Then:** diagnoses 테이블에 레코드가 생성되고, `status`는 기본값 'processing', `deadline_mode`는 기본값 false, `created_at`이 자동 설정된다.

**Scenario 2: 데드라인 모드 Diagnosis 생성**
- **Given:** User 레코드가 존재함
- **When:** `prisma.diagnosis.create({ data: { userId: user_id, mode: "couple", deadlineMode: true, deadline: "2026-06-01", filters: { ... } } })`를 실행함
- **Then:** `deadline_mode`가 true, `deadline`이 '2026-06-01'로 저장된다.

**Scenario 3: FK 무결성 위반 시 에러**
- **Given:** 존재하지 않는 `user_id` = "non-existent-uuid"
- **When:** 해당 user_id로 Diagnosis 생성을 시도함
- **Then:** Prisma가 FK constraint violation 에러를 던지며, 레코드가 생성되지 않는다.

**Scenario 4: 사용자 삭제 시 CASCADE 삭제 확인**
- **Given:** User와 연결된 Diagnosis 레코드 3건이 존재함
- **When:** `prisma.user.delete({ where: { id: user_id } })`를 실행함
- **Then:** User 레코드와 함께 연결된 Diagnosis 3건이 모두 CASCADE 삭제된다.

**Scenario 5: JSONB filters 필드 정합성**
- **Given:** 비어있는 진단 테이블
- **When:** `filters` 필드에 `DiagnosisFilters` 인터페이스와 일치하는 JSON 객체를 저장함
- **Then:** JSON이 정확히 저장되고, 조회 시 같은 구조로 반환된다. `filters.addressA.lat` 등 중첩 필드 접근이 정상 동작한다.

**Scenario 6: 상태 전환 확인**
- **Given:** `status` = 'processing'인 Diagnosis 레코드가 존재함
- **When:** `prisma.diagnosis.update({ where: { id }, data: { status: "completed" } })`를 실행함
- **Then:** `status`가 'completed'로 변경되고, 이전 값 'processing'은 더 이상 조회되지 않는다.

## :gear: Technical & Non-Functional Constraints
- **JSONB 호환성:** `filters` 필드는 `@db.JsonB` 어노테이션 사용. SQLite에서는 `Json` 타입으로 TEXT 저장. SQLite ↔ PostgreSQL 간 JSON 필드 쿼리(Prisma `path` 필터) 비호환 가능성을 주석으로 명시.
- **인덱스 전략:** MVP 단계에서는 기본 인덱스만 적용. B-Tree 인덱스가 기본. JSONB GIN 인덱스는 쿼리 패턴이 확정된 후 추가.
- **날짜 처리:** `deadline` 필드는 `@db.Date` (시각 정보 없이 날짜만 저장). Prisma `DateTime?` 타입으로 매핑되나, 클라이언트에서는 YYYY-MM-DD 문자열로 처리.
- **보안 (REQ-NF-017):** `filters.addressA`, `filters.addressB`에 직장 주소가 포함됨. SEC-001 태스크에서 AES-256 암호화 적용 예정 — 이를 TODO 주석으로 명시.
- **Cascade 정책:** User 삭제 시 Diagnosis CASCADE 삭제. 프로덕션에서는 Soft Delete 전환 검토 필요 (v1+).

## :checkered_flag: Definition of Done (DoD)
- [ ] `prisma/schema.prisma`에 Diagnosis 모델이 SRS §6.2.2와 1:1 매핑됨
- [ ] DiagnosisStatus enum 타입 정의 완료
- [ ] `src/types/diagnosis.ts`에 DiagnosisFilters 타입 정의 완료
- [ ] `npx prisma migrate dev` 성공, SQL 파일 생성 확인
- [ ] FK 관계(User → Diagnosis) CASCADE 삭제 동작 확인
- [ ] JSONB `filters` 필드 저장/조회 정상 동작 확인
- [ ] 인덱스 3개 (`userId`, `status`, `[userId, status]`) 생성 확인
- [ ] Prisma Studio에서 User-Diagnosis 관계 CRUD 검증 완료

## :construction: Dependencies & Blockers
- **Depends on:** DB-002 (USER 테이블 — `user_id FK → User.id`)
- **Blocks:**
  - DB-004 (SHARE_LINK 테이블 — `diagnosis_id FK → Diagnosis.id`)
  - API-002 (Diagnosis 도메인 DTO)
  - CMD-DIAG-004 (진단 결과 서버 저장)
  - QRY-DIAG-001 (진단 결과 조회)
  - CMD-DL-001 (데드라인 모드 활성화)
  - CMD-SINGLE-001 (싱글 모드 진단)
  - CMD-CRON-001 (급매 매물 배치 적재)
  - SEC-001 (개인정보 AES-256 암호화 — filters 필드 내 직장 주소)
