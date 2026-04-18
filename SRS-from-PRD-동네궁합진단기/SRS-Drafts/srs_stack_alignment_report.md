# SRS v0.1 → C-TEC 스택 정렬 변경 보고서

> **문서:** SRS_v0.1_opus.md → Rev 1.2 (2026-04-16)
> **목적:** MVP 관점에서 Next.js 풀스택 아키텍처(C-TEC-001~007)에 부합하도록 SRS 전면 갱신

---

## 1. 변경 섹션 요약

| 섹션 | 변경 유형 | 핵심 내용 |
|---|---|---|
| **§1.2.3 Constraints** | 추가 | CON-09~15 기술 스택 제약사항 (C-TEC-001~007) 추가 |
| **§1.2.3 CON-05** | 수정 | 월 비용 500만+300만 → 인프라 50만+API 100만 (Vercel+Supabase) |
| **§3.1.1 폴백 원칙** | 수정 | Datadog 헬스체크 → Vercel 모니터링 + Sentry |
| **§3.2 Client Apps** | 수정 | SPA(React) → Next.js App Router (RSC, Tailwind, shadcn/ui) |
| **§3.3 API Overview** | 전면 수정 | REST `/api/v1/` → Server Actions + Route Handlers, JWT → NextAuth Session |
| **§3.4.1~3 시퀀스** | 수정 | 참여자: 백엔드 API → Next.js Server Action, DB → Prisma+Supabase |
| **§4.2.1 성능 NFR** | 수정 | 측정 도구: Datadog → Vercel Analytics + Sentry Performance |
| **§4.2.2 가용성 NFR** | 수정 | Datadog → Vercel Status + Sentry, 백업 → Supabase 자동 백업 |
| **§4.2.4 비용 NFR** | 수정 | CloudWatch → Vercel/Supabase Dashboard, 목표 대폭 하향 |
| **§4.2.6 모니터링 NFR** | 수정 | Datadog APM → Vercel Analytics + Sentry, CloudWatch → Vercel Dashboard |
| **§4.2.7 확장성 NFR** | 수정 | Vercel 자동 스케일링, Prisma schema 기반 확장 명시 |
| **§6.1 API Endpoint** | 전면 수정 | 구현 방식 컬럼 추가, Server Action/Route Handler 구분, API-12 Cron 추가 |
| **§6.2 ERD 주석** | 추가 | Prisma ORM + SQLite/Supabase 전환 주석 추가 |
| **§6.6 Component Diagram** | **전면 재작성** | SPA→Next.js, API GW 제거, Redis 제거, Vercel Cron, AI Layer 추가 |
| **Rev 노트** | 갱신 | Rev 1.1 → Rev 1.2, 변경 이력 추가 |

---

## 2. ⚠️ 주의사항 보완 방법

### 2.1 PDF 생성 (REQ-FUNC-023)

| 항목 | 기존 | 변경 |
|---|---|---|
| **방식** | 미지정 (puppeteer 암시) | `@react-pdf/renderer` 서버사이드 생성 |
| **근거** | Vercel Serverless Function에서 puppeteer는 메모리/타임아웃 제약 | React 컴포넌트 기반 PDF → 경량, Node.js 네이티브 실행 |
| **제약** | Vercel Pro 기준 1024MB RAM, 60s timeout | A4 1~2쪽 PDF → 충분한 여유 |

### 2.2 급매 크롤링 파이프라인 (REQ-FUNC-016, REQ-NF-005)

| 항목 | 기존 | 변경 |
|---|---|---|
| **방식** | 별도 크롤링 파이프라인 (배치 서버 암시) | **Vercel Cron Jobs** (Pro Plan) |
| **엔드포인트** | 미지정 | `POST /api/cron/crawl-listings` (API-12 신규) |
| **주기** | 4시간 | `0 */4 * * *` cron expression |
| **제약** | Vercel Pro 필요 ($20/월) | MVP 비용 내 충분 (CON-05 월 50만 이내) |
| **타임아웃** | 미지정 | 60초 (Pro), 필요시 Enterprise 300초 |

---

## 3. 기능적 커버리지 최종 확인

| 기능 | REQ IDs | 새 스택 구현 | 상태 |
|---|---|---|---|
| F1: 두 동선 교차 진단 | FUNC-001~008 | Server Action + 카카오 API | ✅ |
| F2: 배우자 공유 링크 | FUNC-009~014 | Server Action + SSR Route | ✅ |
| F3: 데드라인 모드 | FUNC-015~020 | Server Action + Vercel Cron | ✅ |
| F4: 싱글 모드 | FUNC-021~024 | Server Action + @react-pdf | ✅ |
| F5: 입력값 저장·재탐색 | FUNC-025~028 | Server Action + Prisma | ✅ |
| F6: 인증 | FUNC-029 | NextAuth.js (카카오/네이버) | ✅ |
| F7: 결제 | FUNC-030 | Server Action + Route Handler | ✅ |
| LLM 통합 | C-TEC-005,006 | Vercel AI SDK + Gemini | ✅ 신규 |

> **결론: 기능적 손실 0건.** 모든 MVP 기능이 새 스택으로 완전히 커버됩니다.

---

## 4. 미변경 항목 (후속 검토 권장)

| 항목 | 사유 |
|---|---|
| §6.3 상세 시퀀스 다이어그램 (6건) | 참여자명 구버전 유지 (기능 동일, cosmetic 변경) |
| §6.7 Class Diagram | 도메인 모델은 스택 무관, 변경 불요 |
| §6.4 Validation Plan | 비즈니스 검증 플랜은 스택 무관 |
| §5 Traceability Matrix | 요구사항 ID 변경 없으므로 유지 |
