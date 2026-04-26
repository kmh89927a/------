---
title: "Single 도메인"
category: entities
tags: [기술, mvp]
sources: [src-srs, src-task-list]
created: 2026-04-26
updated: 2026-04-26
status: active
---

# Single 도메인 (domain-single)

## 도메인 개요

싱글 모드(F4)의 학군·가족 항목 자동 숨김, 야간 안전 등급(A~D) 조회, 리포트 저장(window.print())을 담당하는 도메인. DB-009/DB-010 제거 후 정적 JSON 에셋(public/data/) 패턴을 적용하며, 수도권 90% 커버리지 CI 검증이 후속 과제.

## 포함 태스크 (3개)

| Task ID | 1줄 요약 |
|---|---|
| CMD-SINGLE-001 | 싱글 모드 진단 (학군 숨김 + 야간 치안/편의시설 활성) |
| QRY-SINGLE-001 | 야간 안전 등급 A~D 조회 (정적 JSON 기반) |
| CMD-SINGLE-002 | 리포트 저장 (window.print() + CSS @media print) |

## 핵심 NFR

| NFR ID | 내용 |
|---|---|
| REQ-NF-010 | PDF 저장 ≤ 1초 |
| REQ-NF-035 | Sentry 에러 추적 |

## 의존성 관계

- **의존**: Diagnosis (CMD-DIAG-002), Foundation (DB-003)
- **피의존**: Test (TEST-006)

## 관련 페이지

- Entities: [[diagnosis]]
- Sources: [[src-srs]]
- Concepts: [[single-mode]], [[architecture-patterns]], [[srs-v1.6-changes]], [[known-follow-ups]], [[task-domains-overview]]
