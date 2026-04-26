---
title: "ShareLink 도메인"
category: entities
tags: [기술, mvp]
sources: [src-srs, src-task-list]
created: 2026-04-26
updated: 2026-04-26
status: active
---

# ShareLink 도메인 (domain-sharelink)

## 도메인 개요

배우자 공유 링크(F2)의 생성, 열람, 비밀번호 검증, 만료 처리를 담당하는 도메인. UUID v4 기반 링크 생성, SSR 공유 리포트 열람, 무료 미리보기 1곳 분리(splitForPreview), 비밀번호 bcrypt 검증(DUMMY_HASH Timing Attack 방지), 회원가입 유도 모달(UI-008)을 포함한다.

## 포함 태스크 (5개)

| Task ID | 1줄 요약 |
|---|---|
| CMD-SHARE-001 | 공유 링크 생성 (UUID v4, 만료 30일, 선택 비밀번호) |
| CMD-SHARE-002 | viewCount 증가 |
| CMD-SHARE-003 | 만료 링크 접근 시 안내 페이지 + 재생성 알림 |
| CMD-SHARE-004 | 비밀번호 검증 (bcrypt + DUMMY_HASH) |
| QRY-SHARE-001 | SSR 공유 리포트 열람 (무료 미리보기 1곳) |

## 핵심 NFR

| NFR ID | 내용 |
|---|---|
| REQ-NF-003 | SSR 로딩 p95 ≤ 2,000ms |
| REQ-NF-006 | 링크 생성 ≤ 500ms |
| REQ-NF-020 | bcrypt 비밀번호 검증 |
| REQ-NF-021 | 비인가 접근 차단 |

## 의존성 관계

- **의존**: Foundation (DB-004, API-003), Diagnosis (CMD-DIAG-004)
- **피의존**: Test (TEST-003, TEST-004)

## 관련 페이지

- Entities: [[share-link-entity]]
- Sources: [[src-srs]]
- Concepts: [[share-link]], [[architecture-patterns]], [[known-follow-ups]], [[task-domains-overview]]
