---
title: "SavedSearch 엔터티"
category: entities
tags: [데이터, mvp]
sources: [src-srs]
created: 2026-04-23
updated: 2026-04-23
status: active
---

# SavedSearch — 저장된 검색 엔터티

## 정의

사용자의 마지막 진단 입력 조건을 저장하는 엔터티. 사용자당 1건만 유지(UPSERT)하며, 재방문 시 이전 조건을 복원하는 데 사용.

## 속성 (Prisma Schema)

| 필드 | 타입 | 설명 |
|---|---|---|
| `userId` | UUID (PK, FK → User) | 사용자 ID (1:1) |
| `searchParams` | JSON | 저장된 검색 조건 (주소, 필터 등) |
| `savedAt` | DateTime | 저장 시각 |

## 관계

```
User 1 ──→ 0..1 SavedSearch (PK = userId)
```

## 비즈니스 규칙

1. **Best effort 저장**: 세션 종료/앱 종료 시 `beforeunload` 이벤트로 저장 시도. 실패해도 사용자에게 미통지.
2. **불러오기**: 재방문 시 "이전 조건 불러오기" 버튼 → 주소·필터 폼 자동 채움 (≤ 1초)
3. **Geocoding 재검증**: 불러온 주소가 유효한지 카카오 Geocoding으로 재확인. 실패 시 재입력 안내.
4. **MVP 축소**: 비교 뷰(FUNC-026), 시나리오 비교(FUNC-027), 행정동 매핑(FUNC-028)은 v1 이후 연기.

## 관련 페이지

- Entities: [[user]], [[diagnosis]]
- Sources: [[src-srs]]
- Concepts: [[saved-search-concept]]
