# UX_FLOW.md — 핵심 UX 시나리오

> **대상 프로토타입:** `Ondayuiprototypefigma`  
> **기반 기획:** `firebase-ui-prompt.md` · SRS v1.6  
> **작성일:** 2026-04-27

---

## 페르소나

| 페르소나 | 설명 | 주요 사용 모드 |
|---------|------|--------------|
| **박지현 (30, 직장인)** | 남자친구와 함께 이사할 동네를 찾는 커플 | 커플 모드 |
| **김민수 (27, 1인 가구)** | 직장 + 주말 카페 거점 근처로 이사 희망 | 싱글 모드 |
| **이수진 (35, 기혼)** | 계약 만료 전 신속 이사 계획 수립 필요 | 마감일 모드 |
| **공유 링크 수신자** | 지인에게 공유받은 진단 결과를 보는 비회원 | 공유 리포트 |

---

## 시나리오 1 — 커플 모드 (메인 플로우)

### 목표
두 직장의 중간 지점 기준으로 커플이 함께 살기 적합한 동네를 찾는다.

### 흐름도

```mermaid
flowchart TD
    A([앱 진입]) --> B[LoginPage]
    B -->|카카오/네이버 OAuth| C[DiagnosisPage]
    B -->|로그인 없이 체험하기| C

    C --> C1["내 직장 주소 입력\n(addressA)"]
    C --> C2["배우자 직장 주소 입력\n(addressB)"]
    C --> C3["모드 선택: 💑 커플"]
    C1 & C2 & C3 --> C4{주소 모두 입력?}
    C4 -->|No| C4A["진단 시작 버튼 비활성\n(disabled)"]
    C4 -->|Yes| C5["진단 시작 클릭"]

    C5 --> D[ResultPage]
    D --> D1["후보 동네 카드 리스트\n(기본 최대 8개)"]
    D --> D2["필터 패널\n(통근시간 · 예산 · 출근시간대)"]
    D --> D3["MapView\n(Mock 지도 + 마커)"]

    D2 -->|슬라이더 조절| D4{후보 존재?}
    D4 -->|No| D5["EmptyState\n(조건 완화 제안 2개)"]
    D5 -->|완화 버튼 클릭| D2
    D4 -->|Yes| D1

    D1 -->|카드 클릭| E["NeighborhoodSheet 열림\n(동네 상세 · 통근 · 시세)"]
    D3 -->|마커 클릭| E
    E -->|닫기| D1

    D -->|공유 버튼| F["UUID 생성\n→ 클립보드 복사\n→ Toast 알림"]
    F --> G["/share/:uuid 링크 생성"]
```

### 주요 인터랙션 포인트

| 단계 | 컴포넌트 | UX 고려사항 |
|------|----------|------------|
| 주소 입력 | `Input` | 자동완성 미구현 (실 서비스에서 카카오 주소 검색 API 연동 필요) |
| 이전 조건 불러오기 | `Button` | Mock 하드코딩 값 사용 (실 서비스: localStorage/서버 저장) |
| 필터 조절 | `Slider` × 2 | 실시간 반응, debounce 없음 (후보 수 변화 즉시 표시) |
| 마커 클릭 | `MapView` | 카드 클릭과 동일한 `select(id)` 호출로 Sheet 동기화 |
| Sheet 표시 위치 | `NeighborhoodSheet` | 모바일: 하단 슬라이드 업 / 데스크탑(md+): 우측 400px |

---

## 시나리오 2 — 싱글 모드

### 목표
1인 가구가 직장 + 여가 거점(카페·운동 등)을 모두 고려해 동네를 찾는다.

### 흐름도

```mermaid
flowchart TD
    A([앱 진입]) --> B[LoginPage]
    B --> C[DiagnosisPage]
    C -->|싱글 모드 선택| X["SinglePage\n(Dev Nav 직접 접근 또는 싱글 라우트)"]

    X --> X1["직장 주소 입력"]
    X --> X2["여가 거점 1 입력 (선택)"]
    X --> X3["여가 거점 2 입력 (선택)"]
    X --> X4["레이어 토글\n🌙 야간치안 · 🏪 편의시설 · ☕ 카페"]

    X1 & X2 & X3 --> X5{최소 1개 입력?}
    X5 -->|No| X5A[진단 시작 비활성]
    X5 -->|Yes| X6[진단 시작 클릭]

    X6 --> X7{비수도권 키워드\n포함?}
    X7 -->|Yes| X7A["오류 toast\n(현재 수도권만 지원)"]
    X7 -->|No| X8["selectCandidates\n(직장 ↔ 여가거점1 교집합 지역)"]

    X8 --> X9["CandidateCard 리스트"]
    X9 -->|야간치안 ON| X10["SafetyGradeBadge\n(A/B/C/D + 텍스트)"]
    X9 -->|편의시설 ON| X11["편의시설 수 표시"]
    X9 -->|카페 ON| X12["카페 수 표시"]

    X9 -->|리포트 저장 클릭| X13["window.print()\n브라우저 인쇄/PDF"]
```

### 레이어 토글 UX

```mermaid
stateDiagram-v2
    [*] --> 야간치안ON : 기본값 ON
    야간치안ON --> 야간치안OFF : Toggle 클릭
    야간치안OFF --> 야간치안ON : Toggle 클릭

    [*] --> 편의시설OFF : 기본값 OFF
    편의시설OFF --> 편의시설ON : Toggle 클릭

    [*] --> 카페OFF : 기본값 OFF
    카페OFF --> 카페ON : Toggle 클릭

    note right of 야간치안ON
        CandidateCard에
        SafetyGradeBadge 표시
    end note
```

---

## 시나리오 3 — 마감일 모드

### 목표
이사 계약 만료일이 정해진 사용자가 역산 타임라인으로 준비 일정을 확인한다.

### 흐름도

```mermaid
flowchart TD
    A[DeadlinePage 접근] --> B["Calendar 표시\n(date-fns ko locale)"]
    B --> C{날짜 선택}
    C -->|오늘~D+6 선택 시도| C1["disabled — 최소 7일 후만 선택 가능"]
    C -->|D+7 이후 선택| D["선택된 날짜 기준 D-Day 카운트"]

    D --> E["타임라인 카드 5개 자동 생성"]
    E --> E1["📦 매물 탐색 (D-30)"]
    E --> E2["🤝 가계약 (D-21)"]
    E --> E3["📝 본계약 (D-14)"]
    E --> E4["💰 잔금 준비 (D-7)"]
    E --> E5["🚚 이사 (D-Day)"]

    E1 & E2 & E3 & E4 & E5 --> F{"각 단계\n상태 판단"}
    F -->|daysUntilStep < 0| FA["Badge: 완료 (outline)"]
    F -->|daysUntilStep ≤ 3| FB["Badge: 긴급 D-N (destructive 🔴)"]
    F -->|그 외| FC["Badge: 예정 D-N (secondary)"]
```

### 상태 배지 시각적 구분

| 상태 | 조건 | 배지 variant | 색상 |
|------|------|-------------|------|
| 완료 | 해당 일자 지남 | `outline` | 회색 |
| 긴급 | D-3 이내 | `destructive` | 빨강 |
| 예정 | 그 외 | `secondary` | 회색 |

---

## 시나리오 4 — 공유 링크 유입 (비회원 전환 플로우)

### 목표
공유받은 링크로 진입한 비회원이 첫 번째 후보 동네를 확인하고, 추가 정보를 보기 위해 회원가입을 유도한다.

### 흐름도

```mermaid
flowchart TD
    A(["공유 링크 클릭\n/share/:uuid"]) --> B[SharePage 렌더]

    B --> C{URL에 addressA/B\n포함 여부}
    C -->|있음| D["selectCandidates 정상 실행"]
    C -->|없음| D2["Demo 기본값 사용\n(강남/여의도)\n+ 데모 배너 표시"]
    D & D2 --> E

    E["후보 카드 리스트"] --> E1["index 0 — 전체 공개\n(CandidateCard 정상)"]
    E --> E2["index 1+ — blur + Lock\n(CandidateCard isLocked=true)"]

    E2 -->|클릭| F["ConversionModal 표시\n(Dialog)"]
    F -->|나중에| G["모달 닫힘 → SharePage 유지"]
    F -->|회원가입하기| H["navigate('/login')"]

    E1 -->|읽기 완료| I["하단 CTA 버튼\n(sticky bottom)"]
    I -->|클릭| H
    H --> J[LoginPage → 가입 후 전체 결과 접근]
```

### 전환 퍼널

```
공유 링크 클릭 (100%)
    ↓
SharePage 접속 (100%)
    ↓
1번 카드 열람 (100%)
    ↓
잠금 카드 클릭 / CTA 클릭 (전환 관심도 측정 포인트)
    ↓
모달 → 회원가입 클릭 (전환율 목표 지점)
    ↓
LoginPage → 가입 완료 → ResultPage (최종 전환)
```

---

## 전체 페이지 상태 전이도

```mermaid
stateDiagram-v2
    [*] --> LoginPage
    LoginPage --> DiagnosisPage : 로그인/게스트 진입
    DiagnosisPage --> ResultPage : 커플 모드 진단 시작
    DiagnosisPage --> SinglePage : 싱글 모드 선택
    ResultPage --> SharePage : 공유 링크 생성
    SharePage --> LoginPage : 회원가입 유도
    LoginPage --> DiagnosisPage : 재진입

    ResultPage --> ResultPage : 필터 조절 (내부 상태 변화)
    SinglePage --> SinglePage : 레이어 토글 (내부 상태 변화)
    DeadlinePage --> DeadlinePage : 날짜 선택 (내부 상태 변화)

    DiagnosisPage --> DeadlinePage : Dev Nav 이동
    DiagnosisPage --> SinglePage : Dev Nav 이동
```

---

## UX 개선 사항 (실 서비스 연동 시)

| 항목 | 현재 (프로토타입) | 개선 방향 |
|------|-----------------|----------|
| 주소 입력 | 자유 텍스트 | 카카오 주소 검색 API (자동완성 + 지도 PIN) |
| 이전 조건 불러오기 | 하드코딩 Mock | Supabase DB 또는 localStorage 연동 |
| 지도 | SVG Mock | react-kakao-maps-sdk 실제 지도 |
| 공유 링크 | UUID + URL query | UUID → 서버 파라미터 복원 (Supabase) |
| 회원 상태 | 없음 (바이패스) | Supabase Auth (Google/Kakao OAuth) |
| 싱글 모드 | 단일 교집합 계산 | 직장+여가거점 N개 가중 중심점 계산 |
