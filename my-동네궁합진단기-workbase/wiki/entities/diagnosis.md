---
title: "Diagnosis 엔터티"
category: entities
tags: [데이터, mvp]
sources: [src-srs]
created: 2026-04-23
updated: 2026-04-23
status: active
---

# Diagnosis — 진단 엔터티

## 정의

사용자의 동선 교차 진단 결과. 두 주소(또는 싱글 모드의 직장+여가)를 입력하면 교차 연산을 통해 후보 동네를 산출하고, 그 결과를 저장하는 핵심 엔터티.

## 속성 (Prisma Schema)

| 필드 | 타입 | 설명 |
|---|---|---|
| `id` | UUID (PK) | 진단 고유 ID |
| `userId` | UUID (FK → User) | 소유 사용자 |
| `deadline` | DateTime? | 데드라인 모드 시 마감일 |
| `status` | String | "processing" \| "completed" \| "expired" |
| `filters` | JSON | 사용자 필터 조건 (통근시간, 예산 등) |
| `mode` | String | "couple" \| "single" |
| `deadlineMode` | Boolean | 데드라인 모드 활성화 여부 |
| `createdAt` | DateTime | 생성일 |

## 관계

```
User 1 ──→ N Diagnosis
Diagnosis 1 ──→ N ShareLink
```

## 비즈니스 규칙

1. 교차 연산은 Client Component에서 Promise.all 병렬로 수행 (Vercel 10초 Timeout 회피)
2. 결과 저장은 Server Action을 통해 Prisma로 DB에 기록
3. 교집합 0곳 시 조건 완화 제안 ≥ 2개 표시 (REQ-FUNC-008)
4. 수도권(서울/경기/인천) 외 주소 입력 시 차단 (REQ-FUNC-031)

## 관련 페이지

- Entities: [[user]], [[share-link-entity]]
- Sources: [[src-srs]]
- Concepts: [[two-route-intersection]], [[deadline-mode]], [[single-mode]]
