# SRS (Software Requirements Specification) 검토 결과서

**대상 문서:** `/SRS-Drafts/SRS_v0.1_opus.md`
**비교 문서 (PRD):** `/PRD_동네궁합진단기_v0.1.md`
**검토 일자:** 2026-04-15

---

## 📋 검토 요약 (Checklist)

| 점검 항목 | 결과 | 비고 |
|:---|:---:|:---|
| 1. PRD의 모든 Story·AC가 SRS의 REQ-FUNC에 반영됨 | ✅ **충족** | Story 3-1 ~ 3-5 및 모든 AC/예외케이스 완벽 반영 |
| 2. 모든 KPI·성능 목표가 REQ-NF에 반영됨 | ✅ **충족** | 북극성/보조 KPI 및 응답시간 목표치 NFR로 전환 완료 |
| 3. API 목록이 인터페이스 섹션에 모두 반영됨 | ✅ **충족** | 3.3 및 6.1 섹션에 외부/내부 API 완벽 매핑 및 인증 추가 |
| 4. 엔터티·스키마가 Appendix에 완성됨 | ✅ **충족** | 6.2 데이터 모델에 설계 디테일 추가하여 반영됨 |
| 5. Traceability Matrix가 누락 없이 생성됨 | ✅ **충족** | Section 5에 매트릭스 형태로 쌍방향 추적성 표기됨 |
| 6. UseCase(mermaid), ERD, CLD, Component Diagram 포함 여부 | ✅ **충족** | UseCase(6.5), ERD(6.2.0), Component(6.6), Class Diagram(6.7) 모두 추가 완료 |
| 7. Sequence Diagram 3~5개가 포함됨 | ✅ **충족** | 총 9개의 Sequence Diagram 작성됨 (3.4, 6.3 섹션) |
| 8. SRS 전체가 ISO 29148 구조를 준수함 | ✅ **충족** | ISO 표준 목차(Intro, Stakeholders, Context 등) 준수 |

---

## 🔍 상세 검토 내용

### 1. PRD의 모든 Story·AC가 SRS의 REQ-FUNC에 반영됨 (✅ 충족)
- **점검 내용**: PRD의 "3. 사용자 스토리와 수용 기준" (Story 3-1 ~ 3-5, 총 17개 AC와 9개 부정 케이스) 반영 여부 확인.
- **결과**: SRS 문서 4.1 "Functional Requirements" 부분의 `REQ-FUNC-001` 부터 `REQ-FUNC-031` 등에 누락 없이 전부 전사 및 고도화 되었습니다. PRD의 4-1에 명시된 주요 우선순위(Must/Should/Could) 추가 기능 또한 빠짐없이 Requirement 항목에 모두 반영되었습니다.

### 2. 모든 KPI·성능 목표가 REQ-NF에 반영됨 (✅ 충족)
- **성능 목표**: PRD "5-1 성능"에 나열된 서비스 응답속도, 로딩시간 등의 p95 목표치들이 SRS의 `REQ-NF-001` ~ `REQ-NF-010` 항목에 측정 방법과 도구 명세와 함께 반영되었습니다.
- **KPI 지표**: PRD의 북극성 KPI(유료 완료 수)부터 7종류의 보조 KPI까지 전부 SRS 4.2.5 "비즈니스 지표(KPI-Driven NFR)" (`REQ-NF-026` ~ `REQ-NF-034`) 파트에 비기능 요구사항 형태로 정확하게 정의되었습니다.

### 3. API 목록이 인터페이스 섹션에 모두 반영됨 (✅ 충족)
- **점검 내용**: 외부 및 내부 API 매핑 확인.
- **결과**: PRD "6-2 외부 API" 및 "6-3 내부 API"가 SRS의 3.1과 3.3 "System Context and Interfaces"에 잘 구조화되었고, 6.1 "API Endpoint List"에서 호출 메서드, Body 필드, 인증 정보 등 구체적인 스펙을 명세하여 완벽히 고도화되었습니다. 특히 인증/결제를 위한 상세 API까지 추가된 점이 훌륭합니다.

### 4. 엔터티·스키마가 Appendix에 완성됨 (✅ 충족)
- **점검 내용**: 시스템 데이터 모델 보존 및 상세화 확인.
- **결과**: PRD의 중심 엔터티(USER, DIAGNOSIS, CANDIDATE_AREA 등)가 각각의 식별자, 데이터 타입 구조, 제약조건, 외래키(Foreign Key) 관계와 함께 SRS의 6.2 "Entity & Data Model" 파트 상세 스키마 테이블로 충실히 서술 및 완성되었습니다.

### 5. Traceability Matrix가 누락 없이 생성됨 (✅ 충족)
- **점검 내용**: 개별 Requirement의 근간이 되는 근거 추적 확인.
- **결과**: SRS 문서 내 Section 5 전체에 걸쳐 5.1(Story ↔ REQ 매핑), 5.2(PRD기능 ↔ REQ 매핑), 5.3(KPI ↔ REQ 매핑), 5.4(Risk ↔ REQ 매핑) 표 형태로 나뉘어 양방향 추적이 완벽히 문서화되었습니다. 표준에 입각한 아주 우수한 추적 매트릭스입니다.

### 6. 핵심 다이어그램 반영 (✅ 충족)
- **점검 내용**: UseCase, ERD, CLD(Class Diagram), Component Diagram 등 필수 다이어그램 명시 여부.
- **결과 (보완 완료)**:
  - **UseCase Diagram** (§6.5): 6개 액터(맞벌이 부부, 맹모삼천지교, 긴급 이사자, 반복 이사자, 이직 후 이사, 배우자)와 16개 UseCase 간의 관계를 mermaid `flowchart`로 시각화. include/extend 관계 표기 및 UseCase ↔ REQ-FUNC 매핑 테이블 포함.
  - **ERD Diagram** (§6.2.0): PRD의 6개 엔터티에 PAYMENT·VIEW_LOG를 추가한 총 8개 엔터티의 관계를 mermaid `erDiagram`으로 표현. PK/FK/UK 및 필드 타입 명시. 6.2.1~6.2.8 스키마 테이블 앞에 배치하여 전체 데이터 구조를 한눈에 파악 가능.
  - **Component Diagram** (§6.6): Client Layer → API Gateway → Backend Services → Core Engine → Data Store → External Systems → Observability 계층을 mermaid `flowchart`로 시각화. 컴포넌트별 역할 및 관련 REQ 매핑 테이블 포함.
  - **CLD (Class Diagram)** (§6.7): 7개 도메인 엔터티 클래스(User, Diagnosis, CommutePoint, CandidateArea, ShareLink, SavedSearch, Payment, ViewLog) + 서비스 클래스(DiagnosisService, ScoringEngine) + 어댑터 패턴(TransportAdapter ← Kakao/Naver) + 6개 Enum을 mermaid `classDiagram`으로 표현. 속성·메서드·참조 관계·다중성(multiplicity) 완비.

### 7. Sequence Diagram 3~5개가 포함됨 (✅ 충족)
- **결과**: 3.4 섹션 (핵심 플로우 관점)에서 3개, 6.3 섹션 (디테일/예외처리 관점)에서 6개로, 총 9개의 Sequence Diagram이 `mermaid` 코드로 삽입되어 기준인 3~5개를 초과 만족하고 있습니다.

### 8. SRS 전체가 ISO 29148 구조를 준수함 (✅ 충족)
- **결과**: ISO/IEC/IEEE 29148:2018 표준의 구조인 `1. Introduction`, `2. Stakeholders`, `3. System Context and Interfaces`, `4. Specific Requirements`, `5. Traceability Matrix`, `6. Appendix` 등 권장 템플릿의 핵심 목차 구조를 성공적으로 준수하며 서술되었습니다.

---

## ✅ 보완 완료 이력

| 보완 항목 | 추가 위치 (SRS 섹션) | 상태 |
|:---|:---|:---:|
| UseCase Diagram | §6.5 UseCase Diagram | ✅ 완료 |
| ERD Diagram | §6.2.0 ERD (Entity Relationship Diagram) | ✅ 완료 |
| Component Diagram | §6.6 Component Diagram | ✅ 완료 |
| CLD (Class Diagram) | §6.7 Class Diagram (CLD) | ✅ 완료 |

> 위 4건의 다이어그램이 SRS Rev 1.1에 반영되어 검토 항목 8건 전체 충족(8/8) 달성.
