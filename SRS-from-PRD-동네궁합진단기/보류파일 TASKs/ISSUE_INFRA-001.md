---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[Infra] INFRA-001: Next.js 15+ App Router 프로젝트 초기화 + Vercel 배포 파이프라인 구성"
labels: 'infra, devops, priority:critical'
assignees: ''
---

## :dart: Summary
- **기능명:** [INFRA-001] Next.js 15+ App Router 프로젝트 초기화 + Vercel 배포 파이프라인 구성
- **목적:** 전체 시스템의 기술 기반(Foundation)을 확립한다. SRS CON-09(C-TEC-001)에 따라 **Next.js App Router 기반의 단일 풀스택 프레임워크**로 프로젝트를 초기화하고, CON-15(C-TEC-007)에 따라 **Vercel 플랫폼에서 Git Push만으로 자동 배포**되는 CI/CD 파이프라인을 구성한다. 이 태스크는 DB, API, UI, 보안, 모니터링 등 후속 94개 전체 태스크의 루트 의존성(Root Dependency)이다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS 제약사항: [`/TASKS/SRS_v1.md#1.2.3 Constraints`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — CON-09 ~ CON-15 (C-TEC-001 ~ C-TEC-007)
- SRS 클라이언트 명세: [`/TASKS/SRS_v1.md#3.2 Client Applications`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — CLT-01 (Next.js 15+ RSC)
- SRS 컴포넌트 다이어그램: [`/TASKS/SRS_v1.md#6.6 Component Diagram`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — Vercel Platform 전체 구조
- SRS 비용 제약: [`/TASKS/SRS_v1.md#4.2.4`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — REQ-NF-024 (월 인프라 비용 무료 ~ 10만원)

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] **T1.** `npx -y create-next-app@latest ./` 로 Next.js 15+ App Router 프로젝트 생성 (TypeScript, ESLint, `src/` 디렉토리 구조)
- [ ] **T2.** 프로젝트 디렉토리 구조 규약 설정:
  ```
  src/
  ├── app/           # App Router (Page, Layout, Route Handlers)
  │   ├── api/       # Route Handlers
  │   ├── (auth)/    # 인증 관련 페이지 그룹
  │   ├── diagnosis/ # 진단 관련 페이지
  │   └── share/     # 공유 링크 SSR 페이지
  ├── actions/       # Server Actions
  ├── components/    # UI 컴포넌트 (shadcn/ui 기반)
  ├── lib/           # 유틸리티, DB client, 외부 API client
  ├── types/         # TypeScript 타입/DTO 정의
  └── styles/        # 글로벌 스타일
  ```
- [ ] **T3.** `.env.local` 템플릿 파일 생성 — 환경변수 키 목록 정의 (값은 비워둠):
  - `DATABASE_URL` (Prisma)
  - `NEXTAUTH_SECRET`, `NEXTAUTH_URL`
  - `KAKAO_CLIENT_ID`, `KAKAO_CLIENT_SECRET`
  - `NAVER_CLIENT_ID`, `NAVER_CLIENT_SECRET`
  - `KAKAO_MOBILITY_API_KEY`
  - `TOSS_PAYMENTS_SECRET_KEY`, `TOSS_PAYMENTS_CLIENT_KEY`
  - `GOOGLE_GENERATIVE_AI_API_KEY`
  - `SENTRY_DSN`
- [ ] **T4.** GitHub 리포지토리 생성 + Vercel 프로젝트 연결 (Git Push → 자동 빌드·배포 확인)
- [ ] **T5.** Vercel 환경변수 설정 가이드 문서화 (Production / Preview / Development 분리)
- [ ] **T6.** `vercel.json` 기본 설정 — Cron Job 스케줄 플레이스홀더, 리전(icn1 = 서울), 헤더 보안 설정
- [ ] **T7.** 초기 `next.config.ts` 설정:
  - `experimental.serverActions` 활성화 확인
  - `images.remotePatterns` 설정 (카카오맵 타일 등)
  - `headers()` 보안 헤더 (X-Frame-Options, CSP 기본)
- [ ] **T8.** 빈 페이지(`/`) 배포 후 Vercel 대시보드에서 빌드 성공 + 200 OK 확인

## :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: 프로젝트 초기화 완료**
- **Given:** 빈 작업 디렉토리가 주어짐
- **When:** `npx create-next-app@latest`으로 프로젝트를 생성하고 `npm run dev`를 실행함
- **Then:** 로컬 `http://localhost:3000`에서 Next.js App Router 기반 페이지가 정상 렌더링된다. TypeScript 컴파일 에러 0건.

**Scenario 2: Vercel 자동 배포 확인**
- **Given:** GitHub 리포지토리에 Vercel 프로젝트가 연결된 상태
- **When:** `main` 브랜치에 커밋을 Push함
- **Then:** Vercel에서 자동 빌드가 시작되고, 빌드 성공 후 프로덕션 URL에서 200 OK 응답을 반환한다. 빌드 시간 ≤ 120초.

**Scenario 3: 환경변수 분리 확인**
- **Given:** `.env.local` 파일에 `DATABASE_URL=local_value`가 설정됨
- **When:** `process.env.DATABASE_URL`을 Server Action에서 참조함
- **Then:** 로컬 환경에서는 `.env.local` 값이, Vercel Production에서는 Vercel Dashboard에 설정된 값이 각각 사용된다.

**Scenario 4: 디렉토리 구조 검증**
- **Given:** 프로젝트가 초기화된 상태
- **When:** `src/app/`, `src/actions/`, `src/lib/`, `src/types/`, `src/components/` 경로를 확인함
- **Then:** 모든 규약 디렉토리가 존재하며, 각 디렉토리에 `.gitkeep` 또는 `index.ts` placeholder가 포함되어 있다.

## :gear: Technical & Non-Functional Constraints
- **기술 스택 강제 (C-TEC-001):** Next.js App Router 단일 풀스택. 프론트·백엔드 분리 금지.
- **배포 단일화 (C-TEC-007):** Vercel 플랫폼만 사용. 별도 CI/CD(Jenkins, GitHub Actions 등) 설정 금지.
- **비용 (REQ-NF-024):** Vercel 무료(Hobby) 또는 Pro 플랜 월 $20 이하. 무료 티어 제약(Serverless 10초 Timeout, 100GB 대역폭) 의식 필수.
- **리전:** `icn1` (서울) 우선 설정. Vercel Edge Function 사용 시에도 아시아 리전 강제.
- **보안 헤더:** `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: strict-origin-when-cross-origin` 기본 적용.

## :checkered_flag: Definition of Done (DoD)
- [ ] `npm run build` 에러 0건, `npm run lint` 경고 0건으로 빌드 성공
- [ ] Vercel Production 배포 URL에서 200 OK 응답 확인
- [ ] `.env.local.example` 파일에 모든 필수 환경변수 키가 문서화됨
- [ ] 프로젝트 디렉토리 구조가 규약과 100% 일치함
- [ ] `vercel.json`에 리전(icn1) 및 보안 헤더 기본 설정 반영됨
- [ ] README.md에 로컬 개발 환경 셋업 가이드 작성됨

## :construction: Dependencies & Blockers
- **Depends on:** None (최상위 루트 태스크)
- **Blocks:**
  - DB-001 (Prisma 프로젝트 초기화)
  - INFRA-002 (Supabase DB 프로비저닝)
  - INFRA-004 (Tailwind + shadcn/ui 설정)
  - INFRA-005 (Vercel AI SDK 설정)
  - SEC-001, SEC-002, SEC-003 (보안 태스크 전체)
  - MON-001 ~ MON-004 (모니터링 태스크 전체)
  - *간접적으로 94개 전체 태스크를 Block*
