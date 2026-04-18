---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[API Contract] API-008: 토스페이먼츠 PG 연동 인터페이스 정의 (결제 요청/콜백/서명 검증 타입)"
labels: 'api, contract, backend, priority:high'
assignees: ''
---

## :dart: Summary
- **기능명:** [API-008] 토스페이먼츠 PG 연동 인터페이스 정의 (결제 요청/콜백/서명 검증 타입)
- **목적:** SRS §3.1 EXT-06 및 §6.3.6 결제 플로우에 따라 **토스페이먼츠 PG사** 연동에 필요한 TypeScript 타입 체계를 확립한다. initiateCheckout Server Action(API-09)과 POST /api/payment/webhook Route Handler(API-10)에서 사용할 결제 요청·결제 결과·웹훅 콜백·HMAC 서명 검증 인터페이스를 정의한다. PCI-DSS 준수를 위해 카드 정보는 타입에 포함하지 않는다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 외부 시스템 EXT-06: [`/TASKS/SRS_v1.md#3.1`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — 토스페이먼츠 (PCI-DSS 준수)
- SRS 결제 요구사항: [`/TASKS/SRS_v1.md#4.1.6`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — REQ-FUNC-030 (1회 30,000원 + 월정액 10,000원)
- SRS 결제 시퀀스: [`/TASKS/SRS_v1.md#6.3.6`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — initiateCheckout → PG → webhook (서명 HMAC 검증)
- SRS API 명세: [`/TASKS/SRS_v1.md#6.1 API-09/10`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md)
  - API-09: `initiateCheckout()` — 요청: diagnosis_id, plan, amount / 응답: payment_id, checkout_url
  - API-10: `POST /api/payment/webhook` — 요청: transaction_id, status, signature / 응답: ack
- SRS 과금 제약: [`/TASKS/SRS_v1.md#1.2.3`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — CON-02 (ADR-002)
- SRS 장애 우회: [`/TASKS/SRS_v1.md#3.1.1`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — EXT-06: 에러 모달 + 수동 재안내
- SRS CLD: [`/TASKS/SRS_v1.md#6.7`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — Payment 클래스 (processCheckout, handleWebhook, refund)

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] **T1.** 결제 요청 타입 정의 (`src/types/payment-pg.ts`):
  ```typescript
  /** 결제 플랜 — DB enum과 일치 */
  export type PgPaymentPlan = 'one_time' | 'subscription';

  /** initiateCheckout Server Action 요청 */
  export interface CheckoutRequest {
    diagnosisId: string;
    plan: PgPaymentPlan;
    amount: number;           // 원 단위 정수
    orderName: string;        // 주문명 (예: "동네 궁합 진단 리포트")
    customerEmail?: string;
    customerName?: string;
    successUrl: string;       // 결제 성공 리디렉션 URL
    failUrl: string;          // 결제 실패 리디렉션 URL
  }

  /** initiateCheckout 응답 */
  export interface CheckoutResponse {
    paymentId: string;        // 내부 Payment UUID
    checkoutUrl: string;      // 토스 결제 페이지 URL
    orderId: string;          // 토스 주문 ID
  }

  /** 결제 금액 상수 — CON-02 (ADR-002) */
  export const PAYMENT_AMOUNTS = {
    ONE_TIME: 30_000,         // 1회 진단 30,000원
    SUBSCRIPTION: 10_000,     // 월정액 10,000원/월
  } as const;
  ```
- [ ] **T2.** 토스페이먼츠 API 응답 타입 (`src/types/payment-pg.ts`):
  ```typescript
  /** 토스페이먼츠 결제 승인 요청 */
  export interface TossPaymentConfirmRequest {
    paymentKey: string;
    orderId: string;
    amount: number;
  }

  /** 토스페이먼츠 결제 승인 응답 */
  export interface TossPaymentConfirmResponse {
    paymentKey: string;
    orderId: string;
    status: TossPaymentStatus;
    totalAmount: number;
    method: string;              // 결제 수단 (카드, 가상계좌 등)
    requestedAt: string;         // ISO 8601
    approvedAt: string;          // ISO 8601
    receipt?: {
      url: string;               // 영수증 URL
    };
  }

  /** 토스 결제 상태 */
  export type TossPaymentStatus =
    | 'READY'
    | 'IN_PROGRESS'
    | 'WAITING_FOR_DEPOSIT'
    | 'DONE'
    | 'CANCELED'
    | 'PARTIAL_CANCELED'
    | 'ABORTED'
    | 'EXPIRED';
  ```
- [ ] **T3.** 웹훅 콜백 타입 정의 (`src/types/payment-pg.ts`):
  ```typescript
  /** POST /api/payment/webhook 수신 페이로드 */
  export interface TossWebhookPayload {
    eventType: 'PAYMENT_STATUS_CHANGED';
    createdAt: string;            // ISO 8601
    data: {
      paymentKey: string;
      orderId: string;
      status: TossPaymentStatus;
      transactionKey: string;     // 트랜잭션 키
      approvedAt?: string;
    };
  }

  /** 웹훅 응답 — SRS API-10 */
  export interface WebhookAckResponse {
    ack: boolean;
  }
  ```
- [ ] **T4.** 서명 검증 인터페이스 정의 (`src/types/payment-pg.ts`):
  ```typescript
  /** HMAC 서명 검증 인터페이스 */
  export interface IPaymentSignatureVerifier {
    /**
     * 토스 웹훅 요청의 서명을 검증.
     * @param payload - 원본 요청 body (문자열)
     * @param signature - 헤더에 포함된 서명값
     * @returns 서명 유효 여부
     */
    verify(payload: string, signature: string): boolean;
  }

  /** 서명 검증 설정 */
  export interface PaymentSecurityConfig {
    webhookSecret: string;        // 토스 웹훅 시크릿 키
    secretKey: string;            // 토스 시크릿 키 (Basic Auth)
    clientKey: string;            // 토스 클라이언트 키
  }
  ```
- [ ] **T5.** PG 에러 타입 정의 (`src/types/payment-pg.ts`):
  ```typescript
  /** 토스페이먼츠 API 에러 응답 */
  export interface TossApiError {
    code: string;                 // 토스 에러 코드
    message: string;              // 에러 메시지
  }

  export class PaymentError extends Error {
    constructor(
      message: string,
      public readonly tossErrorCode?: string,
      public readonly httpStatus: number = 400,
      public readonly retryable: boolean = false,
    ) {
      super(message);
      this.name = 'PaymentError';
    }
  }
  ```
- [ ] **T6.** PG 연동 클라이언트 인터페이스 정의 (`src/types/payment-pg.ts`):
  ```typescript
  /** CLD §6.7 Payment 도메인 서비스 인터페이스 */
  export interface IPaymentService {
    /** 결제 세션 생성 → 토스 checkout URL 반환 */
    createCheckout(req: CheckoutRequest): Promise<CheckoutResponse>;

    /** 결제 승인 (토스 paymentKey 기반) */
    confirmPayment(req: TossPaymentConfirmRequest): Promise<TossPaymentConfirmResponse>;

    /** 웹훅 페이로드 처리 + 서명 검증 */
    handleWebhook(
      payload: string,
      signature: string
    ): Promise<{ success: boolean; paymentId?: string }>;

    /** 결제 취소/환불 */
    cancelPayment(paymentKey: string, reason: string): Promise<boolean>;
  }
  ```
- [ ] **T7.** 설정 상수 정의 (`src/lib/payment-config.ts`):
  ```typescript
  export const PAYMENT_CONFIG = {
    TOSS_API_BASE_URL: 'https://api.tosspayments.com/v1',
    CONFIRM_TIMEOUT_MS: 10_000,
    WEBHOOK_TIMEOUT_MS: 5_000,
    CHECKOUT_RESPONSE_TIMEOUT_MS: 1_000,  // SRS API-09 ≤ 1,000ms
    WEBHOOK_RESPONSE_TIMEOUT_MS: 500,     // SRS API-10 ≤ 500ms
  } as const;
  ```
- [ ] **T8.** 타입 모듈 barrel export (`src/types/index.ts`)에 추가

## :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: CheckoutRequest 타입 안전성**
- **Given:** `CheckoutRequest` 타입이 정의된 상태
- **When:** `amount`에 문자열 "30000"을 전달하여 객체 생성 시도
- **Then:** TypeScript 컴파일 에러 발생 (number 타입 강제).

**Scenario 2: 결제 금액 상수 불변성**
- **Given:** `PAYMENT_AMOUNTS`가 `as const`로 정의된 상태
- **When:** `PAYMENT_AMOUNTS.ONE_TIME = 50000` 재할당 시도
- **Then:** TypeScript 컴파일 에러 발생.

**Scenario 3: IPaymentService 인터페이스 구현 가능성**
- **Given:** `IPaymentService` 인터페이스가 정의된 상태
- **When:** Mock 클래스로 구현
- **Then:** `createCheckout`, `confirmPayment`, `handleWebhook`, `cancelPayment` 4개 메서드가 올바른 시그니처로 구현 가능.

**Scenario 4: 웹훅 페이로드 파싱**
- **Given:** `TossWebhookPayload` 타입이 정의된 상태
- **When:** 토스 웹훅 JSON 문자열을 `JSON.parse` 후 타입 캐스팅
- **Then:** `payload.data.paymentKey`, `payload.data.status` 등 중첩 필드 접근이 타입 안전.

**Scenario 5: PaymentError 에러 분류**
- **Given:** `PaymentError` 클래스가 정의된 상태
- **When:** `new PaymentError("결제 실패", "PAYMENT_FAILED", 400, false)` 생성
- **Then:** `error.tossErrorCode === "PAYMENT_FAILED"`, `error.retryable === false`로 에러 분류 가능.

**Scenario 6: 서명 검증 인터페이스**
- **Given:** `IPaymentSignatureVerifier` 인터페이스가 정의된 상태
- **When:** `verify(payload, signature): boolean` 메서드 구현
- **Then:** 반환 타입이 `boolean`으로 강제됨. 서명 유효/무효를 명확히 구분.

## :gear: Technical & Non-Functional Constraints
- **PCI-DSS 준수:** 카드 번호, CVC, 만료일 등 민감 결제 정보를 타입에 포함하지 않음. 토스 SDK가 처리.
- **서버 전용:** PG 연동 타입은 Server Action/Route Handler에서만 사용. 클라이언트에 `secretKey` 등 노출 금지.
- **금액 정수 (integer):** 모든 금액은 원 단위 정수 (`number` 타입, 소수점 없음).
- **HMAC 서명:** 토스 웹훅은 `Toss-Signature` 헤더에 HMAC-SHA256 서명을 포함. 검증 로직은 CMD-PAY-002에서 구현하되, 타입만 본 태스크에서 정의.
- **응답시간 목표:** SRS API-09 ≤ 1,000ms, API-10 ≤ 500ms — `PAYMENT_CONFIG`에 타임아웃 상수로 반영.

## :checkered_flag: Definition of Done (DoD)
- [ ] `src/types/payment-pg.ts`에 전체 타입 정의 (CheckoutRequest/Response, TossPaymentConfirm, TossWebhookPayload, IPaymentService, IPaymentSignatureVerifier, PaymentError)
- [ ] `src/lib/payment-config.ts`에 설정 상수 정의
- [ ] `PAYMENT_AMOUNTS`에 SRS CON-02 금액(30,000 / 10,000) 반영
- [ ] 민감 결제 정보(카드번호 등) 타입 포함 0건 (PCI-DSS)
- [ ] TypeScript 컴파일 에러 0건
- [ ] `src/types/index.ts` barrel export 업데이트

## :construction: Dependencies & Blockers
- **Depends on:** None (선행 없음, Wave 1 병렬 착수 가능)
- **Blocks:**
  - API-004 (Payment 도메인 DTO — PG 타입 참조)
  - CMD-PAY-001 (결제 요청 initiateCheckout)
  - CMD-PAY-002 (PG 웹훅 처리 + 서명 검증)
  - CMD-PAY-003 (결제 장애 에러 모달)
  - MOCK-003 (결제 Mock 데이터)
