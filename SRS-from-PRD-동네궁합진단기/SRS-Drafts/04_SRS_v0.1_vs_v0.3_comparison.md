# SRS v0.1 vs v0.3 비교 분석 보고서

> **비교 대상:** SRS_v0.1_opus.md ↔ SRS_v0.3_opus.md  
> **분석 일자:** 2026-04-18  
> **최종 갱신:** 2026-04-18 (Rev 1.4 정합성 검수 완료 반영)  
> **분석 관점:** 기술 스택 명확성 · MVP 목표 및 가치전달 조정 · 기타 차이점

---

## 1. 기술 스택의 명확성

v0.3에서는 **Vercel 무료 티어의 실제 제약(10초 Serverless Timeout 등)**을 반영하여 아키텍처 실현 가능성을 높이는 방향으로 기술 스택을 조정했습니다.

| # | 항목 | SRS v0.1 | SRS v0.3 | 변경 의도 |
|---|------|----------|----------|----------|
| 1 | **REQ-FUNC-003 — 교차 진단 연산 위치** | Server Action에서 교통 API 호출 및 교차 연산 수행 | **클라이언트(Client Component)에서 `Promise.all` 비동기 병렬 호출**로 변경. Vercel 무료 티어 10초 Timeout 회피 명시 | Vercel Hobby 플랜의 Serverless Function 10초 제한에 걸리지 않도록 **연산 주체를 브라우저로 이동** |
| 2 | **REQ-NF-001 — 교차 계산 응답 시간** | p95 ≤ **3,000ms** (교통 API 2회 호출 포함) | p95 ≤ **8,000ms** (클라이언트 API 콜 기준) | 클라이언트에서 다수 API를 병렬 호출하는 구조로 전환함에 따라, 현실적 응답 시간 목표로 완화 |
| 3 | **REQ-NF-010 — PDF 리포트 생성 시간** | ≤ 3초 (`@react-pdf/renderer` 서버사이드 생성) | ≤ **1초** (`window.print()` 클라이언트 호출) | 서버 사이드 렌더링 제거 → 브라우저 내장 기능으로 전환. 측정 방법도 서버 로그 → 클라이언트 로그로 변경 |
| 4 | **REQ-NF-011 — 서비스 가용성 (SLA)** | ≥ **99.5%** (월 다운타임 ≤ 3.65시간) | **Best Effort** 지원 (무료 티어 제약 수용) | Vercel 무료 티어에서 SLA 보장 불가능한 현실 반영. 과도한 SLA 약속 제거 |
| 5 | **REQ-NF-014 — RTO (재해 복구 시간)** | RTO ≤ 4시간 | ~~취소선 처리~~ — **MVP 스코프 제외** | 1인 MVP에서 DR Drill은 비현실적. 스코프 아웃 명시 |
| 6 | **REQ-NF-015 — RPO (백업 복구 지점)** | RPO ≤ 1시간 (Supabase 자동 백업) | ~~취소선 처리~~ — **MVP 스코프 제외** | Supabase 무료 플랜에서 자동 백업 보장 범위 제약 반영 |
| 7 | **REQ-NF-039 — 10배 트래픽 수평 확장** | 10,000명/월 수평 확장 가능 (Vercel 자동 스케일링 + Supabase 커넥션 풀링) | ~~취소선 처리~~ — **1차 MVP 스코프 제외** | MVP 단계에서 10x 확장 테스트는 불필요. 실현할 수 없는 약속 제거 |
| 8 | **REQ-NF-040 — API 공급자 24h 전환** | 교통 API 어댑터 패턴으로 24h 이내 전환 가능 | ~~취소선 처리~~ — **1차 MVP 스코프 제외** | MVP에서 멀티 API 어댑터 패턴 구현은 오버엔지니어링. 단순 카카오 1종으로 충분 |
| 9 | **API-11, API-12 엔드포인트** | PDF 다운로드 Route Handler + Cron 크롤링 Route Handler 명시 | **삭제** (API Endpoint List에서 제거) | PDF는 `window.print()`로, 크롤링은 아웃링크로 대체되어 해당 엔드포인트 불필요 |

> [!IMPORTANT]
> v0.3의 핵심 기술 변경은 **"서버 사이드 연산 → 클라이언트 사이드 연산"** 전환입니다. 이는 Vercel 무료 티어의 10초 Timeout 제약을 우회하기 위한 현실적 판단이며, 응답 시간 목표(3초→8초)도 이에 맞춰 완화되었습니다.

---

## 2. MVP 목표 및 가치전달 조정 내용

v0.3에서는 **최소 구현 비용으로 core value를 전달**하는 방향으로 기능 범위와 구현 방식을 조정했습니다.

| # | 항목 | SRS v0.1 | SRS v0.3 | 변경 의도 |
|---|------|----------|----------|----------|
| 1 | **REQ-FUNC-016 — 데드라인 모드 매물 조회** | 서버가 직접 급매 매물을 크롤링·저장하고, 신규 등록 급매를 최상단에 표시 + 경과 시간 표기 | **네이버 부동산 검색 URL을 조합하여 아웃링크로 새 창 열기** (직접 크롤링 대체) | 급매 크롤링의 법적 리스크(robots.txt, 법률 검토)와 구현 복잡도를 회피. **MVP에서는 외부 플랫폼에 매물 탐색을 위임** |
| 2 | **REQ-FUNC-017 — 교집합 매물 필터링** | 서버에서 교집합 매물만 필터링→지도·리스트 동시 표시 (p95 ≤ 1,500ms) | **(조정됨) 매물 직접 필터링 배제** — 아웃링크를 통한 조건 위임으로 대체 | 매물 DB 구축·유지보수 비용 제거. 핵심 가치인 "동선 교차 진단"에만 집중 |
| 3 | **REQ-FUNC-023 — PDF 리포트 저장** | `@react-pdf/renderer` 서버사이드 PDF 생성 (A4 1~2쪽, 생성 ≤ 3초) | **`window.print()` + CSS `@media print`** 제어로 브라우저 기본 PDF 저장 안내 | 서버 사이드 PDF 렌더링 라이브러리 의존성 제거. **브라우저 내장 기능 활용**으로 구현 비용 0 |
| 4 | **ERD — 데이터 모델 간소화** | 8개 엔터티: USER, DIAGNOSIS, COMMUTE_POINT, CANDIDATE_AREA, SHARE_LINK, SAVED_SEARCH, PAYMENT, VIEW_LOG | **4개 엔터티로 축소**: USER, DIAGNOSIS, SHARE_LINK, PAYMENT | COMMUTE_POINT·CANDIDATE_AREA·SAVED_SEARCH·VIEW_LOG 엔터티 제거. **MVP에서 관리할 데이터 복잡도 대폭 축소** |
| 5 | **ERD — 엔터티 관계** | USER→SAVED_SEARCH, DIAGNOSIS→COMMUTE_POINT, DIAGNOSIS→CANDIDATE_AREA, SHARE_LINK→VIEW_LOG 등 복잡한 관계 | 핵심 관계만 유지: USER→DIAGNOSIS, USER→PAYMENT, DIAGNOSIS→SHARE_LINK, DIAGNOSIS→PAYMENT | 불필요한 Join 쿼리 감소, 스키마 마이그레이션 부담 최소화 |
| 6 | **Section 6.2 — 엔터티 상세 정의** | COMMUTE_POINT(6.2.3), CANDIDATE_AREA(6.2.4), SAVED_SEARCH(6.2.6), VIEW_LOG(6.2.8) 섹션 존재 | **전부 삭제** (6.2.3 → SHARE_LINK, 6.2.4 → PAYMENT으로 재넘버링) | 데이터 모델 문서도 실제 구현 스코프에 맞게 정리 |

> [!TIP]
> v0.3의 MVP 가치전달 전략은 **"직접 구현 → 외부 위임"** 패턴입니다. 매물 탐색은 네이버 부동산 아웃링크로, PDF는 브라우저 `print()`로, 데이터 모델은 핵심 4개로 축소하여 **1인 개발자의 구현 부담을 극적으로 줄였습니다.**

---

## 3. 기타 차이점

| # | 항목 | SRS v0.1 | SRS v0.3 | 비고 |
|---|------|----------|----------|------|
| 1 | **문서 크기** | 1,536줄 / 87,491 bytes | **1,450줄 → Rev 1.4에서 추가 축소** | 불필요한 엔터티 정의·API 엔드포인트·다이어그램 클래스 삭제로 경량화 |
| 2 | **API Endpoint List 항목 수** | API-01 ~ **API-12** (12개) | API-01 ~ **API-10** (10개) | API-11(PDF Route Handler), API-12(Cron 크롤링) 삭제 |
| 3 | ✅ **6.3.1 상세 시퀀스 (교차 진단)** | Server Action이 Transport Adapter를 통해 API 호출·스코어링 수행 | **Rev 1.4에서 Client Component 병렬 호출로 전면 재작성** | REQ-FUNC-003 정렬 완료. Server Action은 결과 저장(`saveDiagnosisResult`)만 담당 |
| 4 | ✅ **6.3.4 싱글 모드 시퀀스** | `@react-pdf/renderer` Route Handler로 PDF 서버 생성 | **Rev 1.4에서 `window.print()` + CSS `@media print`로 변경** | REQ-FUNC-023 정렬 완료. PDF·ReactPDF participant 제거 |
| 5 | ✅ **Component Diagram (6.6)** | Report Module에 `@react-pdf/renderer` 명시 | **Rev 1.4에서 `window.print() + CSS @media print` · 리포트 조회(SSR)로 갱신** | Client→ReportMod 연결로 변경. Route Handlers에서 `/api/cron/crawl-listings` 제거, Cron Jobs 설명 간소화, 매물 소스를 네이버 부동산 아웃링크로 변경 |
| 6 | ✅ **Class Diagram (6.7)** | CommutePoint·CandidateArea·SavedSearch·ViewLog 클래스 포함 | **Rev 1.4에서 4개 클래스 삭제, ERD 4엔터티와 정렬** | `DiagnosisService`→`DiagnosisClientService`(클라이언트), `TransportAdapter`(인터페이스)→`KakaoTransportClient`(구현체)로 단순화. SafetyGrade enum 제거 |
| 7 | ✅ **REQ-NF-010 — PDF 생성 NFR** | ≤ 3초 (`@react-pdf/renderer` 서버사이드) | **Rev 1.4에서 ≤ 1초 (`window.print()` 클라이언트) 변경** | 측정 방법도 서버 로그 → 클라이언트 로그로 전환 |
| 8 | **Section 1~3 (Introduction·Stakeholders·Context)** | — | 변경 없음 | Scope, Constraints, Assumptions, External Systems, Client Apps, API Overview 모두 동일 |
| 9 | **Section 4.1 (나머지 Functional Requirements)** | — | 변경 없음 | REQ-FUNC-003, 016, 017, 023 외 나머지 31개 요구사항은 완전 동일 |
| 10 | **Section 4.2.3~4.2.6 (보안·비용·KPI·운영)** | — | 변경 없음 | 비기능 요구사항 중 보안·비용·KPI·운영 영역은 차이 없음 |
| 11 | **Section 5 (Traceability Matrix)** | — | 변경 없음 | Story↔REQ↔TC 매핑, KPI↔REQ 매핑, Risk↔REQ 매핑 모두 동일 |

---

## 종합 평가

> [!NOTE]
> ### v0.3의 변경 방향성 요약
>
> **"1인 AI-assisted MVP 개발에 맞게 기술적 현실성을 확보"**하는 것이 v0.3의 핵심 변경 철학입니다.
>
> 1. **서버 부하 분산:** 무거운 연산을 서버(10초 제한)에서 클라이언트로 이동
> 2. **기능 외부 위임:** 매물 탐색·PDF 생성 등 부가 기능을 외부 플랫폼/브라우저 기능에 위임
> 3. **데이터 모델 간소화:** 8개 엔터티 → 4개로 축소하여 스키마 복잡도 제거
> 4. **비현실적 SLA 제거:** 99.5% SLA, RTO 4h, RPO 1h 등 무료 티어에서 불가능한 약속 삭제

> [!TIP]
> ### ✅ 정합성 검수 완료 (Rev 1.4, 2026-04-18)
>
> 이전 리뷰에서 식별된 4건의 다이어그램 불일치가 SRS v0.3 **Rev 1.4**에서 모두 해결되었습니다:
>
> | # | 불일치 항목 | 수정 내용 | 상태 |
> |---|-----------|---------|------|
> | 1 | **6.3.1 상세 시퀀스 ↔ REQ-FUNC-003** | Server Action 기반 → **Client Component `Promise.all` 병렬 호출**로 전면 재작성. Transport Adapter·Naver·ODsay participant 제거. 최종 응답 목표 ≤ 8초 반영 | ✅ 해결 |
> | 2 | **6.3.4 싱글 모드 시퀀스 ↔ REQ-FUNC-023** | PDF Route Handler·`@react-pdf/renderer` participant 제거 → **`window.print()` + CSS `@media print`** 흐름으로 교체. 응답 ≤ 1초 | ✅ 해결 |
> | 3 | **6.6 Component Diagram ↔ REQ-FUNC-023** | Report Module: `@react-pdf/renderer PDF 생성` → **`window.print() + CSS @media print · 리포트 조회(SSR)`**. Client Components에 교차 진단 연산·PDF 저장 역할 추가. Route Handlers에서 Cron 크롤링 제거. 매물 소스를 네이버 부동산 아웃링크로 변경 | ✅ 해결 |
> | 4 | **6.7 Class Diagram ↔ ERD** | CommutePoint·CandidateArea·SavedSearch·ViewLog **4개 클래스 삭제**. `DiagnosisService` → `DiagnosisClientService`(클라이언트 연산 전담), `TransportAdapter` 인터페이스 → `KakaoTransportClient`(단일 구현체)로 간소화. SafetyGrade enum 제거 | ✅ 해결 |
> | 5 | **REQ-NF-010 ↔ REQ-FUNC-023** | `@react-pdf/renderer` 서버사이드 ≤ 3초 → **`window.print()` 클라이언트 ≤ 1초**. 측정 방법: 서버 로그 → 클라이언트 로그 | ✅ 해결 |

