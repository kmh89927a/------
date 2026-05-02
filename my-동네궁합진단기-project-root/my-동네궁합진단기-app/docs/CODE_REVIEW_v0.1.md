# Ondayuiprototypefigma 코드 품질 평가서 v0.1 (Part 1/3)

**평가일**: 2026-05-02  
**평가 대상 커밋**: `16eea37` (refactor: 컴포넌트 리팩토링 · 문서화 · 프로토타이핑 단계 완료)  
**종합 점수**: 3.1 / 5.0  
**한 줄 평**: 🟡 **조건부 진입 가능** — 프로토타입 대비 우수한 리팩토링이 이루어졌으나, tsconfig 부재·ESLint 미설정·시맨틱 HTML 전무·arbitrary value 남용 등 P0 이슈 해결 없이는 production 진입 불가.

---

## 1. Executive Summary

### 가장 큰 리스크 3가지

1. **tsconfig.json 자체가 존재하지 않음** — TypeScript strict 모드는커녕 컴파일러 설정 자체가 없음. Vite의 esbuild가 TS를 트랜스파일만 하고 타입 체크는 전혀 수행되지 않는 상태.
2. **ESLint / Prettier 설정 완전 부재** — 코드 스타일 강제 수단이 없어 production 팀 협업 시 즉시 품질 저하 예상.
3. **시맨틱 HTML 전무** — 6개 페이지 모두 `<div>` 기반. `<main>`, `<nav>`, `<section>`, `<header>` 사용 0건. MapView 마커가 `<div>` + `onClick`으로 구현되어 키보드 접근 불가.

### 가장 잘 된 점 2가지

1. **문서화 수준 탁월** — 파일명 주석, JSDoc, 함수 호출 구조 주석, README, docs/ 3개 문서, AI_CONTEXT_GUIDE.md, component_tree.md까지 Figma Make 출신 프로토타입으로서는 이례적으로 완성도 높음.
2. **컴포넌트 리팩토링 완성도** — CandidateCard 3→1 통합, CommuteRow·SafetyGradeBadge·EmptyState·LayerToggleRow 추출, DiagnosisContext로 prop drilling 해소 등 재사용 설계가 잘 되어 있음.

### 종합 판정: 🟡 조건부 진입

---

## 2. 차원별 평가

### 2.1 아키텍처 & 폴더 구조 — 3.5/5

- 🟢 **폴더 구조 일관성 양호**
  - **근거**: `src/app/{pages,components,context}` + `src/lib/{types,mocks}` — layer-based 구조가 일관됨.
  - **문제**: 없음. 프로토타입 규모에 적합한 구조.

- 🟡 **pnpm-workspace.yaml이 무의미함**
  - **근거**: `pnpm-workspace.yaml:1-2` — `packages: ['.']` 단일 패키지만 선언.
  - **문제**: workspace의 이점(패키지 분리·공유)이 전혀 활용되지 않는 over-engineering.
  - **권장 조치**: production에서 monorepo 구조가 아니라면 삭제. monorepo라면 `apps/web`, `packages/shared` 등으로 분리.

- 🟡 **package.json 이름이 Figma Make 자동생성 상태**
  - **근거**: `package.json:2` — `"name": "@figma/my-make-file"`
  - **문제**: 실제 프로젝트와 무관한 자동생성 이름.
  - **권장 조치**: `"name": "@onday/ui-prototype"` 등 의미 있는 이름으로 변경.

- 🟡 **react/react-dom이 peerDependencies에만 존재**
  - **근거**: `package.json:73-76` — `react: 18.3.1`, `react-dom: 18.3.1`이 peer로만 선언.
  - **문제**: `npm install` 시 React가 자동 설치되지 않을 수 있음. `package-lock.json`에 의존하는 취약한 구조.
  - **권장 조치**: `dependencies`에 `react`, `react-dom` 명시 추가.

- 🟡 **라우팅에 lazy loading 없음**
  - **근거**: `routes.tsx:14-19` — 6개 페이지 모두 정적 import.
  - **권장 조치**: `React.lazy()` + `<Suspense>` 래핑으로 코드 스플리팅.

### 2.2 타입 안전성 (TypeScript) — 2.0/5

- 🔴 **tsconfig.json이 존재하지 않음**
  - **근거**: 프로젝트 루트 및 하위에 tsconfig 파일 0개 (node_modules 제외).
  - **문제**: `strict: true`, `noUncheckedIndexedAccess`, `noImplicitAny` 등 모든 strict 옵션이 비활성. Vite esbuild는 타입 체크 없이 트랜스파일만 수행하므로, 타입 오류가 빌드에 잡히지 않음.
  - **권장 조치**: 아래 내용으로 `tsconfig.json` 생성:
    ```json
    {
      "compilerOptions": {
        "target": "ES2020",
        "lib": ["ES2020", "DOM", "DOM.Iterable"],
        "module": "ESNext",
        "moduleResolution": "bundler",
        "strict": true,
        "noUncheckedIndexedAccess": true,
        "noUnusedLocals": true,
        "noUnusedParameters": true,
        "jsx": "react-jsx",
        "baseUrl": ".",
        "paths": { "@/*": ["./src/*"] }
      },
      "include": ["src"]
    }
    ```
    `package.json` scripts에 `"typecheck": "tsc --noEmit"` 추가.

- 🟡 **`any` 사용은 0건이나, 타입 캐스팅에 `as` 남용**
  - **근거**: `DiagnosisPage.tsx:67` — `setMode(v as "couple" | "single")`. `ResultPage.tsx:101` — `v as [number, number]`.
  - **문제**: RadioGroup과 Slider의 value 타입이 `string`/`number[]`로 반환되어 캐스팅이 필요한 구조. tsconfig strict 없이는 이 캐스팅의 안전성 검증 불가.
  - **권장 조치**: tsconfig strict 활성화 후, 제네릭 래퍼 또는 타입 가드 함수로 교체.

- 🟢 **도메인 타입 SSOT 분리 우수**
  - **근거**: `lib/types/neighborhood.ts` — `Neighborhood`, `CommuteInfo`, `CandidateFilters` 3개 타입이 JSDoc과 함께 잘 정의됨.
  - **문제**: 없음. re-export 패턴도 적절.

- 🟡 **MapView가 타입을 mocks에서 직접 import**
  - **근거**: `MapView.tsx:2` — `import type { Neighborhood } from "../../lib/mocks/candidates"`
  - **문제**: 타입 SSOT인 `lib/types/neighborhood.ts`가 아닌 mock 파일에서 타입을 가져옴. SSOT 원칙 위반.
  - **권장 조치**: `import type { Neighborhood } from '@/lib/types/neighborhood'`로 변경.

### 2.3 컴포넌트 설계 — 4.0/5

- 🟢 **SRP 준수 및 재사용 설계 우수**
  - **근거**: `shared/` 5개 컴포넌트 모두 단일 책임. `CandidateCard`의 `isLocked/isSelected/layerConfig` variant 분기가 깔끔.
  - **문제**: 없음.

- 🟢 **모든 컴포넌트가 300줄 이하**
  - **근거**: 가장 큰 파일이 `ResultPage.tsx` (179줄), `SinglePage.tsx` (164줄). 모두 300줄 기준 통과.

- 🟢 **Props 인터페이스 명확**
  - **근거**: 모든 shared 컴포넌트에 명시적 interface + JSDoc 주석.

- 🟡 **fontFamily 인라인 스타일 6곳 반복**
  - **근거**: `LoginPage.tsx:9`, `DiagnosisPage.tsx:26`, `ResultPage.tsx:57`, `SharePage.tsx:51,99`, `DeadlinePage.tsx:43`, `SinglePage.tsx:77`, `NeighborhoodSheet.tsx:32`
  - **문제**: `style={{ fontFamily: 'Pretendard Variable, sans-serif' }}`가 모든 페이지 루트에 반복. `theme.css:129`의 body 스타일에 이미 동일 font-family가 선언되어 있으므로 **완전히 불필요한 중복**.
  - **권장 조치**: 7곳의 `style={{ fontFamily: ... }}` 전부 삭제. CSS에서 이미 처리됨.

- 🟡 **`ImageWithFallback` 미사용 컴포넌트**
  - **근거**: `components/figma/ImageWithFallback.tsx` — Figma Make가 생성한 컴포넌트이나 프로젝트 어디에서도 import하지 않음.
  - **권장 조치**: 삭제.

### 2.4 디자인 시스템 일관성 — 2.5/5

- 🔴 **Tailwind arbitrary value(`text-[Npx]`) 다수 남용**
  - **근거**: 사용자 코드에서 발견된 arbitrary 폰트 사이즈:
    - `text-[28px]` — LoginPage.tsx:12,13
    - `text-[24px]` — DiagnosisPage:29, SharePage:55, SinglePage:79, NeighborhoodSheet:38
    - `text-[20px]` — ResultPage:62, SinglePage:132, NeighborhoodSheet:63
    - `text-[18px]` — CandidateCard:70, DeadlinePage:63
    - `text-[16px]` — DeadlinePage:78, NeighborhoodSheet:46
  - **문제**: Tailwind 표준 클래스(`text-2xl`, `text-xl`, `text-lg`, `text-base`)를 사용하지 않고 Figma의 pixel 값을 그대로 가져온 전형적인 Figma Make 안티패턴. 디자인 토큰 체계 무시.
  - **권장 조치**: `text-[28px]`→`text-3xl`, `text-[24px]`→`text-2xl`, `text-[20px]`→`text-xl`, `text-[18px]`→`text-lg`, `text-[16px]`→`text-base`로 전환. 또는 `theme.css`에 커스텀 유틸리티 정의.

- 🟡 **하드코딩 색상 3건**
  - **근거**:
    - `LoginPage.tsx:19` — `bg-[#FEE500]` (카카오 노란색)
    - `LoginPage.tsx:30` — `bg-[#03C75A]` (네이버 초록색)
    - `MapView.tsx:15` — `bg-[#e5e3df]` (지도 배경색)
  - **문제**: 카카오/네이버 브랜드 컬러는 정당한 사용이나, CSS 변수로 토큰화하면 유지보수 용이.
  - **권장 조치**: `theme.css`에 `--kakao-brand: #FEE500`, `--naver-brand: #03C75A` 추가 후 Tailwind 토큰으로 사용. MapView 배경색은 Mock이므로 P2.

- 🟡 **`w-[400px]`, `h-[60vh]`, `h-[70vh]` 등 레이아웃 arbitrary value**
  - **근거**: `NeighborhoodSheet.tsx:31`, `MapView.tsx:15`
  - **문제**: 디자인 토큰 외 하드코딩 크기. 반응형 처리 미흡.
  - **권장 조치**: `w-[400px]` → `w-96` 또는 `max-w-sm`. `h-[60vh]` → CSS custom property.

- 🟢 **테마 토큰 구조는 잘 갖춰져 있음**
  - **근거**: `theme.css` — light/dark 모드 CSS 변수 전체 정의. `@theme inline`으로 Tailwind v4 연동 완료.
  - **문제**: 토큰이 잘 정의되어 있으나 **실제 컴포넌트에서 활용도가 낮음** (arbitrary value 남용).
# Ondayuiprototypefigma 코드 품질 평가서 v0.1 (Part 2/3)

---

### 2.5 접근성 (a11y) — 3.0/5

- 🟢 **aria-label 전반 적용**
  - **근거**: 모든 Button에 `aria-label` 적용. EmptyState에 `role="status"` + `aria-live="polite"`. SafetyGradeBadge에 `role="status"` + 색+텍스트 동시 표시.
  - 이모지 장식 요소에 `aria-hidden="true"` 처리 완료.

- 🔴 **시맨틱 HTML 전무**
  - **근거**: 6개 페이지 전체 검색 결과, 사용자 코드에서 `<main>`, `<nav>`, `<section>`, `<header>`, `<footer>`, `<article>`, `<aside>` 사용 0건. (sidebar.tsx 내 `<main>`은 shadcn 자동생성 코드)
  - **문제**: 모든 페이지가 `<div>` 기반. 스크린리더 사용자가 페이지 구조를 파악할 수 없음.
  - **권장 조치**:
    - 각 페이지 최외곽 `<div>` → `<main>`
    - ResultPage 상단 필터 영역 → `<header>` 또는 `<section aria-label="필터">`
    - DevNav → `<nav aria-label="개발 네비게이션">`
    - DeadlinePage 타임라인 → `<section aria-label="준비 일정">`

- 🔴 **MapView 마커 키보드 접근 불가**
  - **근거**: `MapView.tsx:41-68` — 마커가 `<div onClick={...}>` 으로 구현.
  - **문제**: `<div>`는 포커스를 받지 못하므로 키보드 사용자가 마커를 선택할 수 없음. `tabIndex`, `onKeyDown`, `role="button"` 모두 없음.
  - **권장 조치**: `<div>` → `<button type="button" aria-label={...}>` 으로 변경. 또는 최소한 `role="button"` + `tabIndex={0}` + `onKeyDown` 추가.

- 🟡 **heading 계층 위반**
  - **근거**: `LoginPage.tsx:12-13` — `<h1>내 하루 동선 맞춤</h1><h2>동네 궁합 진단기</h2>` 시각적 분리를 위해 h1/h2를 사용하나 실제로는 하나의 제목.
  - **권장 조치**: 하나의 `<h1>`로 통합하고 줄바꿈은 `<br />` 또는 CSS.

- 🟡 **색 대비 미검증**
  - **근거**: `muted-foreground: #717182` on `background: #ffffff` — contrast ratio ≈ 4.6:1 (AA 통과하나 AAA 미달). 실제 WCAG AA 검증은 미수행.
  - **권장 조치**: production 전환 시 axe 또는 Lighthouse 접근성 검사 필수.

### 2.6 성능 & 빌드 — 2.5/5

- 🔴 **미사용 의존성 대량 포함**
  - **근거**: `package.json:11-65` — 55개 dependencies 중 실제 사용:
    - **사용됨**: react-router, lucide-react, sonner, date-fns, class-variance-authority, clsx, tailwind-merge, tw-animate-css, 일부 radix-ui 패키지
    - **미사용 (코드 내 import 0건)**: `@emotion/react`, `@emotion/styled`, `@mui/icons-material`, `@mui/material`, `@popperjs/core`, `canvas-confetti`, `cmdk`, `embla-carousel-react`, `input-otp`, `motion`, `next-themes`, `react-day-picker`(calendar.tsx에서 사용하나 DeadlinePage만), `react-dnd`, `react-dnd-html5-backend`, `react-hook-form`, `react-popper`, `react-resizable-panels`, `react-responsive-masonry`, `react-slick`, `recharts`, `vaul`
    - 그리고 대량의 radix-ui 패키지 중 accordion, alert-dialog, aspect-ratio, avatar, collapsible, context-menu, dropdown-menu, hover-card, menubar, navigation-menu, scroll-area 등은 shadcn ui/ 폴더에 파일은 있으나 프로젝트에서 실제 사용하지 않음.
  - **문제**: 번들 사이즈 404kB(gzip 128kB). MUI+Emotion만 해도 ~200kB 트리셰이킹 후에도 상당량 포함. Figma Make가 "혹시 모를" 라이브러리를 전부 설치한 전형적 패턴.
  - **권장 조치**: 미사용 패키지 전부 `pnpm remove`. 미사용 `ui/` 컴포넌트도 삭제. 예상 번들 감소 50%+.

- 🟡 **코드 스플리팅 미적용**
  - **근거**: `routes.tsx:14-19` — 모든 페이지 정적 import. `React.lazy` 사용 0건.
  - **권장 조치**: `const LoginPage = React.lazy(() => import('./pages/LoginPage'))` 패턴 + `<Suspense>` 래핑.

- 🟡 **sonner dynamic import 경고**
  - **근거**: `ResultPage.tsx:48,51` — `const { toast } = await import('sonner')` — Vite 경고 발생.
  - **권장 조치**: 상단에 `import { toast } from 'sonner'`로 정적 import 전환.

- 🟢 **vite.config.ts 적절**
  - **근거**: figmaAssetResolver 플러그인, `@` alias, React+Tailwind 플러그인 설정 적절.

- 🟡 **`dist/` 폴더가 커밋됨**
  - **근거**: 루트에 `dist/` 디렉터리 존재.
  - **문제**: 빌드 산출물이 소스 관리에 포함. `.gitignore`에 누락된 것으로 추정.
  - **권장 조치**: `.gitignore`에 `dist/` 추가 후 git에서 제거.

### 2.7 문서화 & 유지보수성 — 4.5/5

- 🟢 **README.md 충실**
  - **근거**: 실행법, 라우트 맵, 디렉터리 구조, UX 시나리오 요약, 코드 품질 요약, Mock 한계, 관련 문서 링크까지 147줄 상세 기술.

- 🟢 **docs/ 문서 3개 실제 활용도 높음**
  - `UX_FLOW.md` — 4개 시나리오 Mermaid 플로우차트 + 상태 전이도
  - `COMPONENT_STRUCTURE.md` — 컴포넌트 계층 + 코드 규모 비교
  - `CODE_QUALITY.md` — 버그 목록, 체크리스트, 번들 분석, 기술 부채

- 🟢 **주석 품질 우수 — "왜"를 설명**
  - **근거**: `SharePage.tsx:27-29` — `[버그 수정] addressA/B가 URL에 없을 때...` 버그 원인과 수정 이유를 설명. `SinglePage.tsx:38-41` — SRS와의 불일치 배경 설명. 단순 "무엇"이 아닌 "왜" 중심.

- 🟡 **ESLint / Prettier 설정 완전 부재**
  - **근거**: 프로젝트 루트에 `.eslintrc*`, `eslint.config.*`, `.prettierrc*` 파일 0개.
  - **문제**: 코드 스타일 강제 수단 없음. 팀 협업 시 일관성 유지 불가.
  - **권장 조치**: `eslint.config.js` + `prettier.config.js` 생성. `package.json`에 `"lint": "eslint ."`, `"format": "prettier --write ."` 추가.

- 🟡 **`guidelines/Guidelines.md`가 템플릿 상태**
  - **근거**: `guidelines/Guidelines.md:1-62` — `Add your own guidelines here` + HTML 주석으로 된 예시만 존재. 실제 가이드라인 없음.
  - **권장 조치**: 실제 디자인 시스템 규칙 기입 또는 파일 삭제.

---

## 3. Figma Make 안티패턴 점검

- [x] **자동생성 이름 정리 완료** — Frame47, Group2 같은 이름 없음. 모든 컴포넌트/변수명이 의미 있음.
- [ ] **인라인 스타일/magic number 제거 — 미완료**
  - `style={{ fontFamily: 'Pretendard Variable, sans-serif' }}` — 7곳 반복 (theme.css에 이미 정의된 font-family의 불필요한 중복)
  - `MapView.tsx:44-47` — `left: calc(50% + ${x}px)`, `top: calc(50% + ${y}px)` — 인라인 좌표 계산 (Mock이므로 P2)
- [ ] **mock 데이터 외부 분리 — 부분 완료**
  - `lib/mocks/candidates.ts` — NEIGHBORHOOD_POOL 32개 + selectCandidates() 로직이 같은 파일에 혼합. 데이터와 로직 분리 권장.
  - `DiagnosisPage.tsx:21-23` — `handleLoadPrevious()`에 하드코딩 주소 `"서울 강남구 테헤란로"` 인라인.
  - `DeadlinePage.tsx:13-19` — `TIMELINE_STEPS` 상수가 컴포넌트 파일 내부 (이 정도는 허용 가능).
- [x] **`any` 제거 — 완료** — src/ 전체에서 `any` 타입 사용 0건.
- [ ] **시맨틱 HTML 적용 — 미완료** — `<main>`, `<nav>`, `<section>` 등 사용 0건.
- [x] **shadcn 컴포넌트 일관성 — 양호** — 직접 만든 Button과 shadcn Button이 혼재하는 경우 없음. 모든 UI 요소가 shadcn/ui 통해 사용.
- [ ] **dead code 제거 — 미완료**
  - `components/figma/ImageWithFallback.tsx` — 프로젝트 내 import 0건.
  - `components/ui/` — 48개 파일 중 실제 사용은 약 16개. 나머지 32개는 Figma Make가 설치한 shadcn 전체 컴포넌트.
  - `package.json` — 55개 dependencies 중 절반 이상 미사용.
# Ondayuiprototypefigma 코드 품질 평가서 v0.1 (Part 3/3)

---

## 4. Pass Criteria 체크리스트 (개선 후 만족해야 할 품질 요건)

Production 빌드 진입 전 다음을 모두 충족해야 합니다.

### Must (P0) — 즉시

- [ ] `tsconfig.json` 생성 + `strict: true` + `noUncheckedIndexedAccess: true` + `noUnusedLocals: true`
- [ ] `package.json` scripts에 `"typecheck": "tsc --noEmit"` 추가, CI 연동
- [ ] `tsc --noEmit` 에러 0건 통과
- [ ] `eslint.config.js` + `prettier.config.js` 생성, `"lint"` / `"format"` 스크립트 추가
- [ ] ESLint + Prettier 무경고 통과
- [ ] 모든 `text-[Npx]` arbitrary 폰트 사이즈 → Tailwind 표준 클래스로 전환 (총 15건)
- [ ] 모든 `style={{ fontFamily: ... }}` 인라인 삭제 (7곳, CSS에서 이미 처리됨)
- [ ] 시맨틱 HTML 적용: 페이지 루트 `<div>` → `<main>`, 필터 영역 → `<section>`, DevNav → `<nav>`
- [ ] MapView 마커 `<div onClick>` → `<button>` (키보드 접근 가능하게)
- [ ] `react`와 `react-dom`을 `dependencies`에 명시 추가
- [ ] `package.json` name `@figma/my-make-file` → 실제 프로젝트명으로 변경
- [ ] `dist/` 폴더 `.gitignore`에 추가 및 git에서 제거

### Should (P1) — 1주 내

- [ ] 미사용 npm 패키지 제거 (MUI, Emotion, react-dnd, react-slick, recharts, canvas-confetti, motion, next-themes, react-hook-form, react-popper, react-resizable-panels, react-responsive-masonry, cmdk, input-otp, vaul, embla-carousel-react 등 약 20개)
- [ ] 미사용 shadcn `ui/` 컴포넌트 파일 삭제 (약 32개)
- [ ] `components/figma/ImageWithFallback.tsx` 삭제
- [ ] `pnpm-workspace.yaml` 삭제 (단일 패키지이므로 불필요) 또는 monorepo 구조 실제 활용
- [ ] `routes.tsx` — 6개 페이지 `React.lazy()` + `<Suspense>` 코드 스플리팅
- [ ] `ResultPage.tsx:48,51` — sonner dynamic import → 정적 import로 전환
- [ ] `MapView.tsx:2` — 타입 import 경로를 `@/lib/types/neighborhood`로 변경 (SSOT 준수)
- [ ] `LoginPage.tsx:12-13` — h1+h2 heading 계층 위반 수정
- [ ] `guidelines/Guidelines.md` — 실제 가이드라인 기입 또는 삭제
- [ ] `candidates.ts` — NEIGHBORHOOD_POOL 데이터와 selectCandidates 로직 파일 분리
- [ ] 카카오/네이버 브랜드 컬러 CSS 변수 토큰화

### Nice (P2) — 여유 시

- [ ] `NeighborhoodSheet.tsx:20-25` — `window.addEventListener('resize')` → `use-mobile.ts` 훅 활용 또는 CSS 미디어쿼리 기반으로 전환
- [ ] `SharePage.tsx:40` — `window.location.search` 직접 접근 → `useSearchParams()` hook 통일
- [ ] `default_shadcn_theme.css` 루트 파일 → `src/styles/` 하위로 이동 또는 삭제 (theme.css와 중복)
- [ ] `component_tree.md`, `AI_CONTEXT_GUIDE.md` — docs/ 폴더로 이동하여 문서 정리
- [ ] 다크모드 실제 토글 구현 (theme.css에 `.dark` 정의는 있으나 토글 UI 없음)
- [ ] `index.html:3` — `lang="en"` → `lang="ko"` 변경 (한국어 서비스)
- [ ] `index.html:7` — title "Interactive Neighborhood Compatibility App" → 한국어 제목으로 변경
- [ ] `<meta name="description">` 추가
- [ ] Lighthouse 접근성 점수 90+ 달성
- [ ] Vitest 단위 테스트 기본 구조 setup

---

## 5. 우선순위 로드맵

| 우선순위 | 작업 | 예상 공수 | 담당 차원 |
|---|---|---|---|
| P0 | tsconfig.json 생성 + strict 활성화 + tsc --noEmit 에러 해결 | 0.5d | 타입 안전성 |
| P0 | ESLint + Prettier 설정 추가 | 0.5d | 유지보수성 |
| P0 | text-[Npx] → Tailwind 표준 클래스 전환 (15건) | 0.25d | 디자인 시스템 |
| P0 | style={{ fontFamily }} 인라인 삭제 (7곳) | 0.1d | 컴포넌트 설계 |
| P0 | 시맨틱 HTML 적용 (main, nav, section 등) | 0.5d | 접근성 |
| P0 | MapView 마커 `<div>` → `<button>` | 0.25d | 접근성 |
| P0 | react/react-dom dependencies 추가 + 패키지명 변경 + dist/ gitignore | 0.1d | 아키텍처 |
| P1 | 미사용 패키지 20개+ 제거 | 0.5d | 성능 |
| P1 | 미사용 ui/ 컴포넌트 32개 삭제 + ImageWithFallback 삭제 | 0.25d | 성능 |
| P1 | React.lazy() 코드 스플리팅 | 0.25d | 성능 |
| P1 | sonner 정적 import + MapView 타입 import 경로 수정 | 0.1d | 컴포넌트·타입 |
| P1 | heading 계층 수정 + guidelines 정리 + 데이터/로직 분리 | 0.25d | 접근성·아키텍처 |
| P2 | 다크모드 토글 + HTML lang/title + meta description + Lighthouse | 0.5d | 접근성·SEO |
| P2 | 테스트 기본 구조 setup | 0.5d | 유지보수성 |

**P0 총 예상 공수: ~2.2일** / P1 총: ~1.35일 / P2 총: ~1일

---

## 6. 차원별 점수 요약

| 차원 | 점수 | 주요 이슈 |
|------|------|-----------|
| 1. 아키텍처 & 폴더 구조 | 3.5/5 | pnpm-workspace 무의미, lazy loading 없음, react peer-only |
| 2. 타입 안전성 | 2.0/5 | **tsconfig.json 부재** (가장 치명적) |
| 3. 컴포넌트 설계 | 4.0/5 | SRP 우수, fontFamily 인라인 반복만 문제 |
| 4. 디자인 시스템 일관성 | 2.5/5 | arbitrary value 15건+ 남용, 토큰 활용 미흡 |
| 5. 접근성 | 3.0/5 | aria-label 양호, **시맨틱 HTML 전무**, MapView 키보드 불가 |
| 6. 성능 & 빌드 | 2.5/5 | 미사용 패키지 20개+, 코드 스플리팅 없음 |
| 7. 문서화 & 유지보수성 | 4.5/5 | 문서 탁월, **ESLint/Prettier 부재** |
| **종합** | **3.1/5** | |

---

## 7. 다음 단계 권장

1. **P0 작업 완료 후 Claude Code 진입** — 특히 tsconfig.json + ESLint는 Claude Code가 생성하는 코드의 품질을 자동으로 검증하는 안전망이므로 반드시 선행.

2. **권장 리팩토링 순서**:
   1. `tsconfig.json` 생성 → `tsc --noEmit` 통과
   2. `eslint.config.js` + `prettier.config.js` 생성 → `npm run lint` 통과
   3. `text-[Npx]` → Tailwind 표준 전환 (기계적 치환, 15건)
   4. `style={{ fontFamily }}` 7곳 삭제 (기계적 삭제)
   5. 시맨틱 HTML + MapView `<button>` 교체
   6. 미사용 패키지·컴포넌트 대량 삭제 (P1이지만 공수 적으므로 함께 처리 권장)
   7. React.lazy 코드 스플리팅

3. **Claude Code에게 전달할 컨텍스트**: 이 평가서의 P0 체크리스트를 "반드시 충족" 제약으로, P1을 "가능하면 충족" 으로 전달하면 production-grade 코드 생성에 효과적.
