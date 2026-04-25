---
title: "기술 스택"
category: concepts
tags: [기술, mvp]
sources: [src-srs, src-implementation-plan]
created: 2026-04-23
updated: 2026-04-23
status: active
---

# 기술 스택 (Tech Stack)

## 정의

동네궁합진단기 MVP의 기술 아키텍처. "C-TEC"(Consolidated Technology) 원칙에 따라 단일 Next.js 풀스택으로 구성.

## 아키텍처 원칙

| # | 원칙 | 설명 |
|---|---|---|
| P-1 | 단일 코드베이스 | 프론트·백 분리 없이 Next.js 단일 프로젝트 |
| P-2 | Server-first | DB 접근은 Server Actions/Route Handlers에서만 |
| P-3 | 인프라 최소화 | Redis, API Gateway, 별도 배치 서버 제거 |
| P-4 | MVP 가치 보존 | 기술 전환이 UX 저하시키지 않아야 |
| P-5 | 비용 현실화 | 월 인프라 ≤ 10만원 |

## 스택 상세

### 코어

| 레이어 | 기술 | 버전/비고 |
|---|---|---|
| 프레임워크 | Next.js (App Router) | 15 |
| ORM | Prisma | SQLite(Dev) / Supabase PG(Prod) |
| 인증 | Supabase Auth | @supabase/ssr, 카카오/네이버 OAuth |
| UI | Tailwind CSS + shadcn/ui | v4 |

### 클라이언트

| 라이브러리 | 용도 |
|---|---|
| react-kakao-maps-sdk | 지도 시각화 |
| @supabase/ssr | 브라우저 인증 클라이언트 |

### 외부 API

| API | 용도 | MVP 사용 |
|---|---|---|
| 카카오 모빌리티 | 교통 데이터 (유일) | ✅ CON-07 |
| 카카오 Geocoding | 주소 → 좌표 | ✅ |
| 국토교통부 | 실거래가 | ✅ |
| 경찰청 | 범죄 통계 (정적 JSON) | ✅ |
| 교육부 | 학교 배정 구역 | ✅ |
| Google Gemini | AI 요약 (선택) | ➖ |
| 네이버 부동산 | 매물 아웃링크 | ✅ |

### 배포·모니터링

| 서비스 | 용도 |
|---|---|
| Vercel | 배포 (무료 티어) |
| Sentry | 에러 추적 |
| Vercel Analytics | 성능 모니터링 |
| Mixpanel/Amplitude | 이벤트 추적 |

## v1.1 → v1.2 제거 목록

| 제거 항목 | 사유 |
|---|---|
| NextAuth.js v5 | → Supabase Auth |
| Toss Payments SDK | → 결제 MVP 제외 |
| Vercel Cron 크롤링 | → 아웃링크 대체 |
| @react-pdf/renderer | → window.print() |
| 네이버/ODsay 교통 API | → 카카오 1종 |

## 관련 페이지

- Entities: [[user]], [[diagnosis]]
- Sources: [[src-srs]], [[src-implementation-plan]], [[src-task-list]]
- Concepts: [[two-route-intersection]]
