---
title: "MVP 구현 로드맵 v1.2"
category: sources
tags: [기술, mvp]
sources: []
created: 2026-04-23
updated: 2026-04-23
status: active
---

# MVP 구현 로드맵 v1.2

## 문서 정보

| 항목 | 값 |
|---|---|
| 원본 파일 | `raw/assets/plan_srs_stack_alignment_implementation_v1.2.md` |
| 버전 | v1.2 |
| 작성일 | 2026-04-21 |
| 크기 | 42,339 bytes / 914 lines |

## 핵심 요약

1인 퍼블리셔급 개발자가 바이브코딩(AI 코딩 도구)으로 49일 내 MVP를 완주하기 위한 Phase별 구현 계획. v1.2에서 크롤링 제거, 서버 PDF 제거, 결제 PG 제거, 교통 API 1종 축소를 통해 56일 → 49일로 단축.

## 주요 내용

### 핵심 전환 원칙 (5대 원칙)

| # | 원칙 |
|---|---|
| P-1 | 단일 코드베이스 (Next.js) |
| P-2 | Server-first (DB 접근은 Server Action에서만) |
| P-3 | 인프라 최소화 (Redis, API Gateway 제거) |
| P-4 | MVP 가치 보존 |
| P-5 | 월 인프라 비용 ≤ 10만원 |

### Phase별 핵심 결정

| Phase | 핵심 결정 |
|---|---|
| Phase 0 | Supabase Auth (NextAuth 대체) |
| Phase 1 | ERD 4엔터티 (Payment/Listing 제거) |
| Phase 2 | Client Component 병렬 교차 연산 (Vercel 10초 Timeout 회피) |
| Phase 3 | WTP 설문 (결제 모달 대체) |
| Phase 4 | 네이버 부동산 아웃링크 (크롤링 대체) |
| Phase 5 | window.print() + CSS @media print (서버 PDF 대체) |
| Phase 6 | SavedSearch UPSERT 1건 (비교 뷰 연기) |

### v1.1 → v1.2 제거 항목

- NextAuth.js v5 → Supabase Auth
- Toss Payments SDK → 결제 MVP 제외
- Vercel Cron 크롤링 → 네이버 아웃링크
- @react-pdf/renderer → window.print()
- 네이버/ODsay 교통 폴백 → 카카오 1종

### 실현 가능성 판정

- 총 기간: 49일 (약 10주)
- 외부 인프라: Vercel + Supabase (2개)
- 월 비용: ≤ 10만원

## 관련 Wiki 페이지

- Sources: [[src-srs]], [[src-task-list]], [[src-prd]]
- Concepts: [[tech-stack]], [[two-route-intersection]]
