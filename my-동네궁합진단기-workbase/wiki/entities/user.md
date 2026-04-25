---
title: "User 엔터티"
category: entities
tags: [데이터, mvp]
sources: [src-srs]
created: 2026-04-23
updated: 2026-04-23
status: active
---

# User — 사용자 엔터티

## 정의

서비스에 OAuth(카카오/네이버)로 인증한 사용자. Supabase `auth.users.id`를 참조하며, 진단(Diagnosis)과 저장된 검색(SavedSearch)의 소유자.

## 속성 (Prisma Schema)

| 필드 | 타입 | 설명 |
|---|---|---|
| `id` | UUID (PK) | Supabase auth.users.id와 일치 |
| `email` | String (unique) | 이메일 |
| `authProvider` | String | "kakao" \| "naver" |
| `mode` | String | "couple" \| "single" (기본: couple) |
| `createdAt` | DateTime | 생성일 |
| `updatedAt` | DateTime | 수정일 |

## 관계

```
User 1 ──→ N Diagnosis (creates)
User 1 ──→ 0..1 SavedSearch (saves)
```

## 비즈니스 규칙

1. 인증은 Supabase Auth를 통해 수행 ([[src-srs]] REQ-FUNC-029)
2. `mode`는 커플 모드와 싱글 모드를 구분 — 진단 시 학군/가족 항목 표시 여부 결정
3. RLS(Row Level Security): `auth.uid()::text = user_id` 조건으로 본인 데이터만 접근

## 관련 페이지

- Entities: [[diagnosis]], [[saved-search]], [[share-link-entity]]
- Sources: [[src-srs]], [[src-implementation-plan]]
- Concepts: [[tech-stack]]
