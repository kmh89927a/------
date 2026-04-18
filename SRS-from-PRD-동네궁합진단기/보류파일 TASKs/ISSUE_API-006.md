---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[API Contract] API-006: 공통 에러 코드 체계 정의"
labels: 'api, contract, backend, frontend, priority:high'
assignees: ''
---

## :dart: Summary
- **기능명:** [API-006] 공통 에러 코드 체계 정의 (HTTP Status + Application Error Code + 사용자 메시지 매핑)
- **목적:** 모든 Server Action, Route Handler, 클라이언트 에러 핸들링이 참조할 **통합 에러 코드 체계(Error Code Registry)**를 확립한다. SRS §4.1 전체 기능의 AC(Acceptance Criteria)에 명시된 에러 시나리오(예: 주소 1개 입력 시 인라인 에러, 비수도권 차단, 과거 날짜 차단, 만료 링크 안내, 결제 실패 등)를 모두 체계적으로 열거하고, HTTP 상태 코드·내부 애플리케이션 에러 코드·사용자 노출 메시지를 3-layer로 매핑한다. 이 태스크는 선행 의존성이 없어 Wave 1에서 병렬 착수할 수 있으며, 후속 모든 Feature 태스크에서 import하여 사용한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS F1 AC (에러 시나리오): [`/TASKS/SRS_v1.md#4.1.1`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — REQ-FUNC-002 (주소 1개 에러), REQ-FUNC-007 (API 타임아웃), REQ-FUNC-008 (교집합 0곳)
- SRS F2 AC: [`/TASKS/SRS_v1.md#4.1.2`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — REQ-FUNC-010 (만료 링크), REQ-FUNC-014 (유료 전환 모달)
- SRS F3 AC: [`/TASKS/SRS_v1.md#4.1.3`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — REQ-FUNC-019 (급매 0건), REQ-FUNC-020 (과거 날짜)
- SRS F4 AC: [`/TASKS/SRS_v1.md#4.1.4`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — REQ-FUNC-024 (비수도권 커버리지)
- SRS F5 AC: [`/TASKS/SRS_v1.md#4.1.5`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — REQ-FUNC-028 (행정동 변경)
- SRS F6: [`/TASKS/SRS_v1.md#4.1.6`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — REQ-FUNC-031 (수도권 외 차단)
- SRS 외부 시스템 장애 우회: [`/TASKS/SRS_v1.md#3.1.1`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — EXT-01 ~ EXT-08 장애 시 우회 전략
- SRS NFR: [`/TASKS/SRS_v1.md#4.2.2`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — REQ-NF-012 (5xx 에러율 ≤ 0.5%)

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] **T1.** 에러 응답 공통 인터페이스 정의 (`src/types/error.ts`):
  ```typescript
  export interface AppError {
    code: string;           // 내부 에러 코드 (예: "DIAG_001")
    httpStatus: number;     // HTTP 상태 코드 (400, 404, 409, 500 등)
    message: string;        // 사용자 노출 메시지 (한국어)
    detail?: string;        // 개발자 디버깅용 상세 메시지 (프로덕션에서 숨김)
  }

  export type AppErrorCode = keyof typeof APP_ERRORS;
  ```
- [ ] **T2.** 도메인별 에러 코드 레지스트리 정의 (`src/lib/errors.ts`):

  **Auth 도메인:**
  | Error Code | HTTP | 사용자 메시지 | SRS 근거 |
  |---|---|---|---|
  | `AUTH_001` | 401 | "로그인이 필요합니다" | REQ-FUNC-029 |
  | `AUTH_002` | 403 | "접근 권한이 없습니다" | REQ-NF-021 |
  | `AUTH_003` | 503 | "로그인 서비스에 일시적 오류가 발생했습니다" | EXT-07 우회 |

  **Diagnosis 도메인:**
  | Error Code | HTTP | 사용자 메시지 | SRS 근거 |
  |---|---|---|---|
  | `DIAG_001` | 400 | "두 번째 주소를 입력해 주세요" | REQ-FUNC-002 |
  | `DIAG_002` | 422 | "해당 지역은 현재 수도권만 지원됩니다" | REQ-FUNC-024, REQ-FUNC-031 |
  | `DIAG_003` | 200 | "조건을 만족하는 동네가 없습니다. 최대 통근 시간을 늘려보세요" | REQ-FUNC-008 |
  | `DIAG_004` | 504 | "일시적 오류가 발생했습니다. 잠시 후 다시 시도해 주세요" | REQ-FUNC-007 |
  | `DIAG_005` | 404 | "진단 결과를 찾을 수 없습니다" | API-02 |

  **ShareLink 도메인:**
  | Error Code | HTTP | 사용자 메시지 | SRS 근거 |
  |---|---|---|---|
  | `SHARE_001` | 410 | "이 링크는 만료되었습니다" | REQ-FUNC-010 |
  | `SHARE_002` | 401 | "비밀번호가 일치하지 않습니다" | REQ-NF-020 |
  | `SHARE_003` | 404 | "존재하지 않는 공유 링크입니다" | REQ-FUNC-011 |
  | `SHARE_004` | 403 | "무료 미리보기를 모두 사용했습니다" | REQ-FUNC-013 |

  **Payment 도메인:**
  | Error Code | HTTP | 사용자 메시지 | SRS 근거 |
  |---|---|---|---|
  | `PAY_001` | 402 | "결제가 필요합니다" | REQ-FUNC-030 |
  | `PAY_002` | 400 | "결제 처리 중 오류가 발생했습니다" | EXT-06 우회 |
  | `PAY_003` | 403 | "웹훅 서명 검증에 실패했습니다" | API-10 서명 검증 |
  | `PAY_004` | 409 | "이미 결제가 완료된 진단입니다" | REQ-FUNC-030 |

  **Deadline 도메인:**
  | Error Code | HTTP | 사용자 메시지 | SRS 근거 |
  |---|---|---|---|
  | `DL_001` | 400 | "마감일은 오늘 이후여야 합니다" | REQ-FUNC-020 |
  | `DL_002` | 200 | "현재 조건의 급매가 없습니다" | REQ-FUNC-019 |

  **SavedSearch 도메인:**
  | Error Code | HTTP | 사용자 메시지 | SRS 근거 |
  |---|---|---|---|
  | `SAVE_001` | 404 | "저장된 탐색 기록을 찾을 수 없습니다" | REQ-FUNC-026 |
  | `SAVE_002` | 200 | "저장된 주소 '○○동'이 변경되었습니다" | REQ-FUNC-028 |

  **공통:**
  | Error Code | HTTP | 사용자 메시지 | SRS 근거 |
  |---|---|---|---|
  | `COMMON_001` | 429 | "요청이 너무 많습니다. 잠시 후 다시 시도해 주세요" | REQ-NF-022 |
  | `COMMON_002` | 500 | "서버 오류가 발생했습니다" | REQ-NF-012 |
  | `COMMON_003` | 503 | "일시적 네트워크 지연이 발생했습니다" | EXT-01 우회 |

- [ ] **T3.** 에러 코드 상수 객체 (`APP_ERRORS`) as const 타입 추론 가능 구조로 export
- [ ] **T4.** 에러 응답 헬퍼 함수 구현 (`src/lib/error-response.ts`):
  ```typescript
  // Server Action용 에러 응답 생성기
  export function createAppError(code: AppErrorCode, detail?: string): AppError

  // Route Handler용 NextResponse 에러 응답 생성기
  export function createErrorResponse(code: AppErrorCode, detail?: string): NextResponse

  // 클라이언트용 에러 토스트 메시지 추출기
  export function getErrorMessage(error: unknown): string
  ```
- [ ] **T5.** 클라이언트(Client Component)에서 에러 코드를 파싱하여 적절한 UI(토스트/인라인 에러/모달)로 분기하는 유틸 타입 정의:
  ```typescript
  export type ErrorDisplayType = 'toast' | 'inline' | 'modal' | 'page';

  export function getErrorDisplayType(code: AppErrorCode): ErrorDisplayType
  ```
- [ ] **T6.** 에러 코드 단위 테스트 — 모든 에러 코드가 유효한 HTTP 상태 코드를 가지는지, 모든 도메인이 누락 없이 커버되는지 검증

## :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: Server Action에서 에러 응답 생성**
- **Given:** `DIAG_001` 에러 코드가 정의된 상태
- **When:** Server Action에서 `createAppError('DIAG_001')`을 호출함
- **Then:** `{ code: "DIAG_001", httpStatus: 400, message: "두 번째 주소를 입력해 주세요" }` 형태의 에러 객체가 반환된다.

**Scenario 2: Route Handler에서 에러 NextResponse 생성**
- **Given:** `SHARE_001` 에러 코드가 정의된 상태
- **When:** Route Handler에서 `createErrorResponse('SHARE_001')`을 호출함
- **Then:** HTTP 410 상태 코드와 함께 `{ error: { code: "SHARE_001", message: "이 링크는 만료되었습니다" } }` 형태의 JSON 응답이 반환된다.

**Scenario 3: 클라이언트 에러 메시지 추출**
- **Given:** Server Action이 `AppError` 형태의 에러를 throw함
- **When:** 클라이언트에서 `getErrorMessage(error)`를 호출함
- **Then:** 사용자에게 표시할 한국어 메시지 문자열이 반환된다. 알 수 없는 에러는 "서버 오류가 발생했습니다" 폴백.

**Scenario 4: 에러 표시 유형 분기**
- **Given:** 에러 코드별 DisplayType이 정의된 상태
- **When:** `getErrorDisplayType('DIAG_001')`을 호출함
- **Then:** `'inline'`이 반환된다 (주소 입력 필드 옆 인라인 에러).

**Scenario 5: 프로덕션 환경 detail 숨김**
- **Given:** `NODE_ENV=production` 환경
- **When:** `createErrorResponse('COMMON_002', 'DB connection timeout')`을 호출함
- **Then:** 응답 JSON에 `detail` 필드가 포함되지 않는다 (디버깅 정보 노출 차단).

**Scenario 6: 에러 코드 완전성 검증**
- **Given:** SRS에 명시된 모든 에러 시나리오 목록
- **When:** `APP_ERRORS` 상수 객체의 키를 열거함
- **Then:** SRS의 모든 에러 시나리오가 1:1로 매핑되어 있으며, 누락된 에러 코드가 0건이다.

## :gear: Technical & Non-Functional Constraints
- **타입 안전성:** `APP_ERRORS` 객체는 `as const` 어설션으로 정의하여 에러 코드를 리터럴 타입으로 추론 가능하게 한다.
- **국제화 준비:** 현재는 한국어 메시지만 정의하되, 향후 i18n 확장을 위해 메시지 키 분리 구조를 주석으로 안내.
- **로깅 연동 (MON-001):** `detail` 필드는 Sentry에 전송되는 로그에 포함. 사용자 응답에서는 프로덕션 시 제거.
- **HTTP 상태 코드 규약:**
  - `200`: 비즈니스 로직상 "결과 없음" (에러는 아니지만 사용자 안내 필요)
  - `400`: 클라이언트 입력 검증 실패
  - `401/403`: 인증/인가 실패
  - `404`: 리소스 없음
  - `409`: 비즈니스 충돌 (중복 결제 등)
  - `410`: 리소스 만료 (공유 링크)
  - `422`: 비즈니스 규칙 위반 (비수도권 등)
  - `429`: Rate Limit 초과
  - `500/503/504`: 서버/외부 서비스 에러

## :checkered_flag: Definition of Done (DoD)
- [ ] `src/types/error.ts`에 `AppError` 인터페이스 정의 완료
- [ ] `src/lib/errors.ts`에 도메인별 에러 코드 레지스트리 (`APP_ERRORS`) 정의 (총 20+개 코드)
- [ ] `src/lib/error-response.ts`에 헬퍼 함수 3개 (`createAppError`, `createErrorResponse`, `getErrorMessage`) 구현
- [ ] 모든 에러 코드에 대한 단위 테스트 통과 (유효한 HTTP 상태 코드, 비어있지 않은 메시지)
- [ ] SRS §4.1 AC에 명시된 모든 에러 시나리오가 코드로 매핑됨 (누락 0건)
- [ ] TypeScript 타입 추론이 정상 동작 — `createAppError('INVALID_CODE')` 시 컴파일 에러

## :construction: Dependencies & Blockers
- **Depends on:** None (선행 의존성 없음, Wave 1 병렬 착수 가능)
- **Blocks:**
  - API-001 ~ API-005 (모든 도메인 DTO — 에러 응답 타입 참조)
  - CMD-DIAG-001 ~ 007 (Diagnosis 도메인 — 에러 핸들링)
  - CMD-SHARE-001 ~ 004 (ShareLink 도메인 — 만료/비밀번호 에러)
  - CMD-PAY-001 ~ 003 (Payment 도메인 — 결제 에러)
  - CMD-DL-001 ~ 003 (Deadline 도메인 — 날짜 검증 에러)
  - CMD-AUTH-001 ~ 004 (Auth 도메인 — 인증 에러)
  - TEST-001 ~ 010 (테스트 태스크 — 에러 시나리오 검증)
  - UI-001 ~ 015 (프론트엔드 — 에러 UI 분기 로직)
