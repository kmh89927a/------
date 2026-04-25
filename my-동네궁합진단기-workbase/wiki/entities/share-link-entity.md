---
title: "ShareLink 엔터티"
category: entities
tags: [데이터, mvp]
sources: [src-srs]
created: 2026-04-23
updated: 2026-04-23
status: active
---

# ShareLink — 공유 링크 엔터티

## 정의

진단 결과를 배우자(비회원)에게 공유하기 위한 URL 토큰. UUID v4 기반이며, 30일 만료 정책을 가짐. 서비스의 바이럴 루프에서 핵심 역할.

## 속성 (Prisma Schema)

| 필드 | 타입 | 설명 |
|---|---|---|
| `id` | UUID (PK) | 링크 고유 ID |
| `diagnosisId` | UUID (FK → Diagnosis) | 대상 진단 |
| `uniqueUrl` | String (unique) | UUID v4 토큰 (entropy ≥ 128bit) |
| `passwordHash` | String? | 열람 비밀번호 해시 (nullable) |
| `viewCount` | Int | 열람 횟수 |
| `freePreviewUsed` | Boolean | 무료 미리보기 1곳 사용 여부 |
| `expiresAt` | DateTime | 만료 시간 (생성일 +30일) |
| `createdAt` | DateTime | 생성일 |

## 관계

```
Diagnosis 1 ──→ N ShareLink
```

## 비즈니스 규칙

1. **비회원 접근**: 앱 설치 없이 SSR 페이지로 리포트 열람 가능 (REQ-FUNC-011)
2. **무료 미리보기**: 최초 1곳만 무료, 2곳째 접근 시 WTP 설문 유도 모달 (REQ-FUNC-013~014)
3. **30일 만료**: 만료 후 개인정보 노출 0건, 재생성 안내 (REQ-FUNC-010)
4. **비밀번호**: 선택적 bcrypt 해시 검증
5. **OG 메타태그**: `generateMetadata()`로 카카오톡 리치 프리뷰 자동 생성

## KPI 연동

- 공유 링크 클릭률 목표: ≥ 40%
- 공유 → 2nd 유저 전환율 목표: ≥ 15%

## 관련 페이지

- Entities: [[diagnosis]], [[user]]
- Sources: [[src-srs]], [[src-prd]]
- Concepts: [[share-link]]
