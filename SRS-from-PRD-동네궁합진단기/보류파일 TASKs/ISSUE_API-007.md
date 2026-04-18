---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[API Contract] API-007: 카카오 모빌리티 API 클라이언트 인터페이스 정의 (KakaoTransportClient 타입)"
labels: 'api, contract, frontend, priority:high'
assignees: ''
---

## :dart: Summary
- **기능명:** [API-007] 카카오 모빌리티 API 클라이언트 인터페이스 정의 (KakaoTransportClient 타입)
- **목적:** SRS §3.1 EXT-01 및 §6.7 CLD에 명시된 **카카오 모빌리티 API** 연동을 위한 TypeScript 타입 체계를 확립한다. 교집합 후보 동네 산출(REQ-FUNC-003)에서 클라이언트 브라우저가 직접 카카오 API를 호출하는 구조(Promise.all 병렬)이므로, Client Component에서 사용할 카카오 API 요청/응답 타입, 에러 타입, 클라이언트 클래스 인터페이스를 정의한다. 이 계약이 MOCK-004(Mock 응답)와 CMD-DIAG-001~006(진단 로직) 모두의 SSOT가 된다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 외부 시스템 EXT-01: [`/TASKS/SRS_v1.md#3.1`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — 카카오 모빌리티 API (입력: 출발/도착 좌표·시각, 출력: 경로·소요시간·환승·도보)
- SRS 제약: [`/TASKS/SRS_v1.md#1.2.3`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — CON-07 (카카오 1종만 사용, 장애 시 에러 모달)
- SRS 교차 진단 시퀀스: [`/TASKS/SRS_v1.md#6.3.1`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — Client Component에서 Kakao API 병렬 호출
- SRS CLD: [`/TASKS/SRS_v1.md#6.7`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — KakaoTransportClient 클래스 (getRoute, getCommuteTime)
- SRS 기능 요구사항: [`/TASKS/SRS_v1.md#4.1.1`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — REQ-FUNC-003 (클라이언트 Promise.all 병렬), REQ-FUNC-004 (출퇴근 시간 ±10%), REQ-FUNC-005 (시간대별 재계산), REQ-FUNC-007 (5초 타임아웃 + 재시도)
- SRS 장애 우회: [`/TASKS/SRS_v1.md#3.1.1`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — EXT-01: 토스트→Retry
- SRS API 제약: ASM-01 — 카카오 모빌리티 API 무료 tier 일 50만 건

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] **T1.** 좌표 및 공통 타입 정의 (`src/types/geo.ts`):
  ```typescript
  export interface Coordinate {
    lat: number;    // 위도
    lng: number;    // 경도
  }

  export interface Address {
    address: string;        // 전체 주소 문자열
    roadAddress?: string;   // 도로명 주소
    coordinate: Coordinate;
    regionCode?: string;    // 법정동 코드
  }
  ```
- [ ] **T2.** 카카오 API 요청/응답 타입 정의 (`src/types/kakao-transport.ts`):
  ```typescript
  /** 카카오 경로 요청 파라미터 */
  export interface KakaoRouteRequest {
    origin: Coordinate;
    destination: Coordinate;
    departureTime?: string;   // HH:mm (예: "08:00")
    priority?: 'RECOMMEND' | 'TIME' | 'DISTANCE';
  }

  /** 카카오 경로 응답 — 개별 경로 */
  export interface KakaoRouteResult {
    totalTime: number;        // 총 소요시간 (분)
    totalDistance: number;     // 총 거리 (m)
    transferCount: number;    // 환승 횟수
    walkDistance: number;     // 도보 거리 (m)
    fare: number;             // 요금 (원)
    departureTime: string;    // 출발 시각
    arrivalTime: string;      // 도착 시각
    legs: KakaoRouteLeg[];    // 구간별 상세
  }

  /** 경로 구간 상세 */
  export interface KakaoRouteLeg {
    mode: 'WALK' | 'BUS' | 'SUBWAY' | 'EXPRESS_BUS' | 'TRAIN';
    sectionTime: number;      // 구간 소요시간 (분)
    distance: number;         // 구간 거리 (m)
    startName?: string;       // 출발 정류장/역명
    endName?: string;         // 도착 정류장/역명
    route?: string;           // 노선명 (예: "2호선", "370번")
  }

  /** 교통수단별 소요시간 요약 */
  export interface CommuteTimeResult {
    transit: KakaoRouteResult | null;   // 대중교통
    car: KakaoRouteResult | null;       // 자차
    origin: Coordinate;
    destination: Coordinate;
    departureTime: string;
  }
  ```
- [ ] **T3.** KakaoTransportClient 인터페이스 정의 (`src/types/kakao-transport.ts`):
  ```typescript
  /** CLD §6.7 KakaoTransportClient 클래스 인터페이스 */
  export interface IKakaoTransportClient {
    /**
     * 출발지→도착지 경로 계산
     * @throws KakaoApiError - API 호출 실패 시
     */
    getRoute(
      origin: Coordinate,
      destination: Coordinate,
      departAt?: string
    ): Promise<KakaoRouteResult>;

    /**
     * 출퇴근 소요시간 조회 (대중교통 + 자차)
     * @throws KakaoApiError - API 호출 실패 시
     */
    getCommuteTime(
      origin: Coordinate,
      destination: Coordinate
    ): Promise<CommuteTimeResult>;
  }
  ```
- [ ] **T4.** 카카오 API 에러 타입 정의 (`src/types/kakao-transport.ts`):
  ```typescript
  export class KakaoApiError extends Error {
    constructor(
      message: string,
      public readonly statusCode: number,
      public readonly kakaoErrorCode?: string,
      public readonly isTimeout: boolean = false,
      public readonly retryable: boolean = false,
    ) {
      super(message);
      this.name = 'KakaoApiError';
    }
  }

  export interface KakaoApiErrorResponse {
    code: number;
    msg: string;
  }
  ```
- [ ] **T5.** API 호출 설정 상수 정의 (`src/lib/kakao-config.ts`):
  ```typescript
  export const KAKAO_API_CONFIG = {
    BASE_URL: 'https://apis-navi.kakaomobility.com',
    TIMEOUT_MS: 5000,          // REQ-FUNC-007: 5초 타임아웃
    MAX_RETRIES: 1,            // REQ-FUNC-007: 자동 재시도 1회
    DAILY_LIMIT: 500_000,      // ASM-01: 무료 tier 일 50만 건
    COMMUTE_HOURS: {
      start: 7,                // REQ-FUNC-005: 오전 7시
      end: 9,                  // REQ-FUNC-005: 오전 9시
    },
    TIME_ACCURACY_THRESHOLD: 0.10,  // REQ-FUNC-004: ±10% 오차
  } as const;
  ```
- [ ] **T6.** 후보 동네 관련 타입 정의 (`src/types/candidate.ts`):
  ```typescript
  /** 교집합 후보 동네 */
  export interface CandidateArea {
    id: string;
    name: string;                         // 행정동명
    regionCode: string;                   // 법정동 코드
    coordinate: Coordinate;
    commuteFromA: CommuteTimeResult;     // 직장A까지 출퇴근 시간
    commuteFromB: CommuteTimeResult;     // 직장B까지 출퇴근 시간
    score: number;                        // 종합 스코어
    rank: number;                         // 순위
  }

  /** 후보 동네 산출 결과 (클라이언트 연산 결과) */
  export interface CandidateResult {
    candidates: CandidateArea[];
    totalCandidatesEvaluated: number;
    computeTimeMs: number;
  }
  ```
- [ ] **T7.** 타입 모듈 barrel export (`src/types/index.ts`)에 추가

## :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: 타입 정합성 검증**
- **Given:** `KakaoRouteRequest` 타입이 정의된 상태
- **When:** TypeScript에서 필수 필드(`origin`, `destination`) 없이 객체 생성 시도
- **Then:** 컴파일 에러 발생. 타입 안전성 확보.

**Scenario 2: KakaoTransportClient 인터페이스 구현 가능성**
- **Given:** `IKakaoTransportClient` 인터페이스가 정의된 상태
- **When:** Mock 클래스로 인터페이스를 구현함
- **Then:** 모든 메서드가 올바른 시그니처로 구현 가능. 반환 타입 Promise 정합성 확인.

**Scenario 3: KakaoApiError 타임아웃 식별**
- **Given:** `KakaoApiError` 클래스가 정의된 상태
- **When:** `new KakaoApiError("Timeout", 408, undefined, true, true)` 생성
- **Then:** `error.isTimeout === true`, `error.retryable === true`로 타임아웃 에러를 프로그래밍적으로 식별 가능.

**Scenario 4: 설정 상수 불변성**
- **Given:** `KAKAO_API_CONFIG`이 `as const`로 정의된 상태
- **When:** `KAKAO_API_CONFIG.TIMEOUT_MS = 10000` 재할당 시도
- **Then:** TypeScript 컴파일 에러 발생. 설정값 불변.

**Scenario 5: CandidateArea 구조 검증**
- **Given:** `CandidateArea` 타입이 정의된 상태
- **When:** 진단 결과 Mock 데이터를 `CandidateArea[]`로 타입 캐스팅
- **Then:** `commuteFromA.transit.totalTime`, `commuteFromB.car.totalTime` 등 중첩 필드 접근이 타입 안전하게 동작.

## :gear: Technical & Non-Functional Constraints
- **클라이언트 실행 (REQ-FUNC-003):** 카카오 API 호출은 Client Component에서 실행. 따라서 타입은 `src/types/`에 정의하되, 서버/클라이언트 양쪽에서 import 가능한 순수 타입 파일이어야 함 (서버 전용 모듈 import 금지).
- **API Key 노출 방지:** 클라이언트에서 카카오 API를 직접 호출하되, API Key는 Next.js `NEXT_PUBLIC_` prefix 환경변수 또는 카카오 JavaScript 앱 키(도메인 제한 설정)를 사용. 서버 사이드 Secret Key는 클라이언트에 노출 금지.
- **단일 API 제약 (CON-07):** 카카오 모빌리티 1종만 사용. 네이버/ODsay 폴백 인터페이스 불필요.
- **오차 기준 (REQ-FUNC-004):** `TIME_ACCURACY_THRESHOLD = 0.10` — 카카오맵 앱 대비 ±10% 이내.

## :checkered_flag: Definition of Done (DoD)
- [ ] `src/types/geo.ts` — Coordinate, Address 타입 정의
- [ ] `src/types/kakao-transport.ts` — KakaoRouteRequest/Result, CommuteTimeResult, IKakaoTransportClient, KakaoApiError 정의
- [ ] `src/types/candidate.ts` — CandidateArea, CandidateResult 타입 정의
- [ ] `src/lib/kakao-config.ts` — API 설정 상수 (`as const`)
- [ ] `src/types/index.ts` barrel export 업데이트
- [ ] TypeScript 컴파일 에러 0건
- [ ] 모든 타입이 서버/클라이언트 양쪽에서 import 가능 (순수 타입, 서버 전용 의존성 없음)

## :construction: Dependencies & Blockers
- **Depends on:** None (선행 없음, Wave 1 병렬 착수 가능)
- **Blocks:**
  - MOCK-004 (카카오 API Mock 응답 데이터)
  - CMD-DIAG-001 (Geocoding 연동)
  - CMD-DIAG-002 (교집합 후보 산출 — Promise.all 병렬)
  - CMD-DIAG-003 (스코어링 엔진)
  - CMD-DIAG-005 (조건 필터 적용)
  - CMD-DIAG-006 (타임아웃 핸들링)
