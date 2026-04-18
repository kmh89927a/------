---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[DB] DB-005: PAYMENT 테이블 Prisma 스키마 정의 및 마이그레이션"
labels: 'database, backend, priority:high'
assignees: ''
---

## :dart: Summary
- **기능명:** [DB-005] PAYMENT 테이블 Prisma 스키마 정의 및 마이그레이션
- **목적:** SRS §6.2.4에 정의된 PAYMENT 엔터티를 Prisma 스키마로 구현한다. PAYMENT 테이블은 하이브리드 과금 모델(1회 30,000원 + 월정액 10,000원/월, ADR-002)의 결제 트랜잭션을 관리한다. USER와 DIAGNOSIS를 FK로 참조하며, 토스페이먼츠 PG사 연동의 결제 상태(pending→success/fail/refunded) 라이프사이클을 추적한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS PAYMENT 데이터 모델: [`/TASKS/SRS_v1.md#6.2.4 PAYMENT`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — 9개 필드 전체 명세
- SRS ERD: [`/TASKS/SRS_v1.md#6.2.0 ERD`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — USER(1:N)→PAYMENT, DIAGNOSIS(1:N)→PAYMENT
- SRS 결제 시퀀스: [`/TASKS/SRS_v1.md#6.3.6`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — initiateCheckout → PG → webhook → Payment UPDATE
- SRS 결제 요구사항: [`/TASKS/SRS_v1.md#4.1.6`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — REQ-FUNC-030 (1회/구독 결제)
- SRS 과금 제약: [`/TASKS/SRS_v1.md#1.2.3`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — CON-02 (ADR-002 하이브리드 과금)
- SRS API 명세: [`/TASKS/SRS_v1.md#6.1 API-09/10`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — initiateCheckout, webhook
- SRS 클래스 다이어그램: [`/TASKS/SRS_v1.md#6.7`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — Payment 클래스, PaymentPlan/PaymentStatus enum
- SRS 비용 KPI: [`/TASKS/SRS_v1.md#4.2.4`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — REQ-NF-025 (유료 리포트 단위 처리 비용 ≤ 3,000원/건)

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] **T1.** `prisma/schema.prisma`에 Payment 관련 Enum 정의:
  ```prisma
  enum PaymentPlan {
    one_time
    subscription
  }

  enum PaymentStatus {
    pending
    success
    fail
    refunded
  }
  ```
- [ ] **T2.** Payment 모델 정의:
  ```prisma
  model Payment {
    id              String        @id @default(uuid())
    userId          String        @map("user_id")
    diagnosisId     String        @map("diagnosis_id")
    plan            PaymentPlan
    amount          Int
    pgTransactionId String?       @map("pg_transaction_id") @db.VarChar(100)
    status          PaymentStatus @default(pending)
    createdAt       DateTime      @default(now()) @map("created_at")
    completedAt     DateTime?     @map("completed_at")

    // Relations
    user            User          @relation(fields: [userId], references: [id], onDelete: Cascade)
    diagnosis       Diagnosis     @relation(fields: [diagnosisId], references: [id], onDelete: Cascade)

    // Indexes
    @@index([userId])
    @@index([diagnosisId])
    @@index([status])
    @@index([pgTransactionId])
    @@map("payments")
  }
  ```
- [ ] **T3.** FK 관계 설정:
  - `userId` → `User.id` (CASCADE)
  - `diagnosisId` → `Diagnosis.id` (CASCADE)
  - User, Diagnosis 모델에 `payments Payment[]` relation 추가 확인
- [ ] **T4.** 인덱스 설정:
  - `userId`: 사용자별 결제 이력 조회
  - `diagnosisId`: 진단별 결제 상태 확인
  - `status`: 상태별 필터링 (pending 건 모니터링 등)
  - `pgTransactionId`: PG사 웹훅에서 트랜잭션 ID로 결제 건 조회
- [ ] **T5.** 마이그레이션 실행: `npx prisma migrate dev --name add-payment-table`
- [ ] **T6.** 마이그레이션 SQL 검증 — SRS §6.2.4 대비 9개 필드 1:1 매핑 확인
- [ ] **T7.** Prisma Studio에서 User ↔ Diagnosis ↔ Payment 3-way 관계 검증

## :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: 1회 결제 레코드 생성**
- **Given:** User, Diagnosis 레코드가 존재함
- **When:** `prisma.payment.create({ data: { userId, diagnosisId, plan: "one_time", amount: 30000 } })`를 실행
- **Then:** payments 테이블에 레코드 생성. `status`='pending', `pgTransactionId`=null, `completedAt`=null.

**Scenario 2: 구독 결제 레코드 생성**
- **Given:** User, Diagnosis 레코드가 존재함
- **When:** `plan: "subscription"`, `amount: 10000`으로 생성
- **Then:** `plan`='subscription', `amount`=10000으로 저장.

**Scenario 3: 결제 상태 전환 (pending → success)**
- **Given:** `status`='pending'인 Payment 레코드 존재
- **When:** `prisma.payment.update({ data: { status: "success", pgTransactionId: "toss_txn_123", completedAt: new Date() } })`
- **Then:** `status`='success', `pgTransactionId`·`completedAt` 업데이트.

**Scenario 4: pgTransactionId로 결제 건 조회**
- **Given:** `pgTransactionId="toss_txn_123"`인 Payment 존재
- **When:** `prisma.payment.findFirst({ where: { pgTransactionId: "toss_txn_123" } })`
- **Then:** 해당 결제 건이 정확히 반환됨. (웹훅 콜백 시나리오)

**Scenario 5: 중복 결제 방지**
- **Given:** `diagnosisId="diag_1"`, `status`='success'인 Payment 존재
- **When:** 동일 `diagnosisId`로 새 Payment 생성 시도
- **Then:** 비즈니스 로직에서 차단 (스키마 레벨 UNIQUE 제약은 아님, 앱 로직에서 처리). 스키마상으로는 생성 가능하나 `PAY_004` 에러로 처리 예정.

**Scenario 6: 환불 상태 전환**
- **Given:** `status`='success'인 Payment 레코드
- **When:** `prisma.payment.update({ data: { status: "refunded" } })`
- **Then:** `status`='refunded'로 변경. `completedAt`은 유지.

## :gear: Technical & Non-Functional Constraints
- **금액 단위:** `amount`는 정수(원 단위). 소수점 없음. 30,000원 = `30000`.
- **PG 트랜잭션 ID:** `pgTransactionId`는 nullable — 결제 요청 시점에는 없고, PG 콜백에서 채워짐.
- **상태 머신:** `pending` → `success` | `fail` → `refunded`. 역방향 전환(`success`→`pending`) 금지 — 비즈니스 로직에서 검증.
- **적세 규정:** PCI-DSS 준수를 위해 카드 정보 일체를 DB에 저장하지 않음. PG사(토스페이먼츠)가 전적으로 처리.
- **비용 추적 (REQ-NF-025):** `amount` 필드를 기반으로 유료 리포트 단위 처리 비용 ≤ 3,000원/건 모니터링.

## :checkered_flag: Definition of Done (DoD)
- [ ] `prisma/schema.prisma`에 Payment 모델이 SRS §6.2.4와 1:1 매핑됨
- [ ] PaymentPlan, PaymentStatus enum 정의 완료
- [ ] `npx prisma migrate dev` 성공
- [ ] FK (User, Diagnosis) CASCADE 삭제 동작 확인
- [ ] 인덱스 4개 생성 확인
- [ ] Prisma Studio에서 3-way 관계(User↔Diagnosis↔Payment) CRUD 검증

## :construction: Dependencies & Blockers
- **Depends on:** DB-002 (USER — `user_id FK`), DB-003 (DIAGNOSIS — `diagnosis_id FK`)
- **Blocks:**
  - API-004 (Payment 도메인 DTO)
  - CMD-PAY-001 (결제 요청 initiateCheckout)
  - CMD-PAY-002 (PG 웹훅 처리)
  - QRY-PAY-001 (결제 이력 조회)
  - MOCK-003 (결제 Mock 데이터)
