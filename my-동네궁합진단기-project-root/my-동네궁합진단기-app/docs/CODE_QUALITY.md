# CODE_QUALITY.md — 코드 품질 평가

> **대상 프로토타입:** `Ondayuiprototypefigma`  
> **평가 기준일:** 2026-04-27  
> **빌드 상태:** ✅ `vite build` 성공 (에러 0, 경고 1)

---

## 종합 평가

| 항목 | 점수 | 근거 |
|------|------|------|
| **중복 제거** | ⭐⭐⭐⭐⭐ | CandidateCard 3→1, CommuteRow 2→1, Switch×3→공유 컴포넌트 |
| **타입 안전성** | ⭐⭐⭐⭐☆ | 타입 SSOT 분리, 모든 props 타입화. `window.location.search` 직접 접근 1건 |
| **문서화** | ⭐⭐⭐⭐⭐ | 파일명 주석 + JSDoc + 함수 호출 구조 + README + docs/ 3개 |
| **상태 관리** | ⭐⭐⭐⭐☆ | Context 도입으로 Drilling 해소. 싱글·공유 페이지는 Context 미사용 |
| **접근성(a11y)** | ⭐⭐⭐⭐☆ | `aria-label` 전반 적용, `aria-live="polite"`, `role="status"`. 포커스 트랩 미구현 |
| **버그 수정** | ⭐⭐⭐⭐⭐ | URL 빈 문자열 버그, 싱글 알고리즘 버그 2건 수정 |
| **빌드 최적화** | ⭐⭐⭐☆☆ | sonner dynamic import 경고 잔존 (기능 영향 없음) |
| **테스트 커버리지** | ⭐☆☆☆☆ | 테스트 코드 없음 (TEST-001~010 별도 태스크로 계획) |

---

## 1. 수정된 버그 목록

### BUG-001 — SharePage URL 빈 문자열 Fallback

| 항목 | 내용 |
|------|------|
| **심각도** | Medium |
| **증상** | `/share/demo123` 직접 접근 시 `addressA=''` → `geocodeAddress('')` → 서울시청 좌표(fallback) → 엉뚱한 동네 노출 |
| **원인** | `useSearchParams()`의 `.get()` 반환값이 `null`일 때 `''` fallback 사용 |
| **수정** | Demo 기본값(`강남/여의도`) + "데모 모드" 배너 표시 |
| **파일** | `src/app/pages/SharePage.tsx` |

### BUG-002 — SinglePage 동일 주소 전달

| 항목 | 내용 |
|------|------|
| **심각도** | Low (의도적 동작처럼 보이나 SRS 스펙과 불일치) |
| **증상** | `selectCandidates(avgAddress, avgAddress)` → 중간점 = 자기 자신 → 커플 모드와 동일한 로직 |
| **원인** | 싱글 모드 특화 로직 미구현 |
| **수정** | `addrA = workplace`, `addrB = leisure1 \|\| workplace`로 분리 |
| **파일** | `src/app/pages/SinglePage.tsx` |

---

## 2. 코드 품질 체크리스트

### 중복 코드 제거

- [x] `CandidateCard` 3개 파일 중복 → `shared/` 추출
- [x] `CommuteRow` 2개 파일 중복 → `shared/` 추출
- [x] `SAFETY_GRADE_CONFIG` 인라인 → `SafetyGradeBadge` 추출
- [x] `Switch+Label` 패턴 ×3 반복 → `LayerToggleRow` 추출
- [x] EmptyState 인라인 → `EmptyState` 추출

### 타입 안전성

- [x] `Neighborhood` · `CommuteInfo` · `CandidateFilters` 타입 SSOT 분리
- [x] 모든 컴포넌트 props 인터페이스 정의
- [x] `CandidateFilters` 중복 선언 제거 (re-export로 통합)
- [ ] `window.location.search` 직접 접근 → `useSearchParams` hook 통일 권장

### 상태 관리

- [x] ResultPage Props Drilling → `DiagnosisContext` 해소
- [x] 필터 초기값 `DEFAULT_FILTERS` 상수화
- [x] `resetFilters()` 함수 제공
- [ ] SinglePage `results` 상태 → Context 미사용 (페이지 단독 상태로 적절)

### 접근성 (a11y)

- [x] 모든 버튼에 `aria-label` 적용
- [x] `EmptyState` `role="status"` + `aria-live="polite"`
- [x] `SafetyGradeBadge` `role="status"` + 색상+텍스트 동시 표시
- [x] `aria-hidden="true"` — 이모지 장식 요소 처리
- [ ] `NeighborhoodSheet` 포커스 트랩 미구현 (shadcn/ui Sheet 내장 처리 확인 필요)
- [ ] 키보드 네비게이션 전체 검증 필요

### 문서화

- [x] 모든 파일 최상단 파일명 주석 (역할·의존 포함)
- [x] 주요 함수/컴포넌트 JSDoc (`@param` · `@returns` · `@example`)
- [x] 복잡한 페이지 파일 함수 호출 구조 주석 (ResultPage · SharePage · SinglePage · DiagnosisContext)
- [x] `candidates.ts` — 알고리즘 단계별 인라인 주석
- [x] README.md 작성
- [x] docs/UX_FLOW.md 작성
- [x] docs/COMPONENT_STRUCTURE.md 작성

### 빌드 / 번들

- [x] `vite build` 에러 0건
- [x] DevNav `import.meta.env.DEV` 가드 (프로덕션 제외)
- [ ] sonner dynamic import 경고 (ResultPage `await import('sonner')`) — 정적 import로 전환 권장
- [ ] 번들 크기: `index.js` 404 kB (gzip 128 kB) — 최적화 여지 있음

---

## 3. 번들 분석

```
dist/assets/index.js    403.99 kB (gzip: 127.64 kB)
dist/assets/index.css    95.14 kB (gzip:  15.27 kB)
```

**주요 번들 구성 요인:**
- Radix UI 전체 (accordion, alert-dialog, avatar... 미사용 컴포넌트 다수 포함)
- MUI (`@mui/icons-material`, `@mui/material`) — 현재 프로토타입에서 미사용
- recharts, react-slick, embla-carousel 등 — 미사용 라이브러리

**권장:** 실 서비스 전환 시 `package.json`에서 미사용 의존성 제거 → 번들 50% 이상 감소 예상.

---

## 4. 기술 부채 목록

| ID | 항목 | 심각도 | 관련 파일 |
|----|------|--------|---------|
| TD-001 | `window.location.search` 직접 접근 | Low | SharePage.tsx |
| TD-002 | sonner dynamic import 경고 | Low | ResultPage.tsx |
| TD-003 | MapView SVG Mock → 실 카카오 지도 | High | MapView.tsx |
| TD-004 | 주소 입력 자유 텍스트 → 카카오 주소 API | High | DiagnosisPage.tsx · SinglePage.tsx |
| TD-005 | 공유 UUID 파라미터 서버 복원 미구현 | Medium | SharePage.tsx |
| TD-006 | 미사용 라이브러리 번들 포함 | Medium | package.json |
| TD-007 | 테스트 코드 전무 | High | 전체 |
| TD-008 | 싱글 모드 N개 여가 거점 가중 계산 | Low | SinglePage.tsx · candidates.ts |

---

## 5. 개선 로드맵

```
Phase 1 (프로토타입 완성)         ← 현재 위치
  ✅ 컴포넌트 리팩토링
  ✅ 버그 수정 2건
  ✅ 문서화 완성

Phase 2 (실 서비스 연동 준비)
  → TD-003: MapView 실 카카오 지도 교체
  → TD-004: 카카오 주소 검색 API 연동
  → TD-007: 테스트 코드 작성 (TEST-001~010)

Phase 3 (서비스 출시 전)
  → TD-001: SharePage useSearchParams 통일
  → TD-002: sonner 정적 import 전환
  → TD-005: UUID → 서버 파라미터 복원
  → TD-006: 미사용 라이브러리 제거
```
