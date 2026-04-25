---
title: "Task List v1.3 — 개발 태스크 목록"
category: sources
tags: [기술, mvp]
sources: []
created: 2026-04-23
updated: 2026-04-23
status: active
---

# Task List v1.3 — 개발 태스크 목록

## 문서 정보

| 항목 | 값 |
|---|---|
| 원본 파일 | `raw/assets/06_TASK_LIST_v1.3.md` |
| 버전 | v1.3 |
| 작성일 | 2026-04-22 |
| 크기 | 37,451 bytes / 506 lines |

## 핵심 요약

SRS v1.6 기반 73개 개발 태스크의 상세 목록. Wave 1(크리티컬 패스) → Wave 2(핵심 기능) → Wave 3(부가 기능) → Wave 4(배포/운영) 순서로 구조화. 각 태스크에 Issue 상태, AC 요약, NFR 매핑이 포함되어 AI 에이전트의 자동 Issue 생성을 지원.

## 주요 내용

### Wave 구조

| Wave | 범위 | 태스크 수 | Phase |
|---|---|---|---|
| Wave 1 | 환경 설정 + DB/Auth | ~20개 | Phase 0-1 |
| Wave 2 | F1 교차 진단 + F2 공유 링크 | ~25개 | Phase 2-3 |
| Wave 3 | F3 데드라인 + F4 싱글 + F5 저장 + AI | ~20개 | Phase 4-7 |
| Wave 4 | 배포 + 운영 + 모니터링 | ~8개 | Phase 8 |

### 크리티컬 패스

```
T-0.1 Next.js 생성
  → T-0.3 Prisma 설정
    → T-1.1 Schema 정의
      → T-2.1 교차 진단 로직
        → T-3.1 공유 링크 Server Action
          → T-8.2 Vercel 배포
```

### AI Agent Guide

- `status: unwritten` 필터로 미작성 태스크 추출
- 각 태스크의 AC(Acceptance Criteria) 요약 포함
- NFR(비기능 요구사항) 매핑으로 성능 기준 명시

## 관련 Wiki 페이지

- Sources: [[src-srs]], [[src-implementation-plan]]
- Concepts: [[tech-stack]]
