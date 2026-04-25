---
title: "두 동선 교차 진단 (F1)"
category: concepts
tags: [mvp, 기술]
sources: [src-srs, src-prd, src-aos-dos]
created: 2026-04-23
updated: 2026-04-23
status: active
---

# 두 동선 교차 진단 (Two-Route Intersection)

## 정의

두 사람(부부)의 직장 주소를 입력받아, 양쪽 통근 시간의 교집합이 최적인 후보 동네를 산출하는 서비스의 **핵심 기능(F1)**. AOS 4.00, DOS 4.00으로 모든 분석에서 1순위.

## 상세

### 알고리즘 흐름

```
1. [Server Action] 두 주소 Geocoding (카카오 Geocoding API) + 수도권 검증
2. [Client]        Transport Adapter 호출 — 양방향 통근시간 계산 (Promise.all 병렬)
3. [Client]        교집합 후보 동네 산출 (스코어링 알고리즘)
4. [Server Action] Prisma로 Diagnosis + 결과 JSON 저장
5. [Client]        결과 렌더링 (지도 + 후보 카드 + 타임라인)
```

### 아키텍처 결정

- **Client Component 연산**: Vercel 무료 티어 10초 Serverless Timeout을 회피하기 위해 외부 교통 API 호출과 교차 연산을 클라이언트에서 수행
- **Transport Adapter 패턴**: 인터페이스를 정의하여 MVP는 카카오 1종만 구현하되, v1 이후 네이버/ODsay 추가 가능
- **에러 핸들링**: API 타임아웃 5초 → 토스트 + 재시도 1회 (FUNC-007)

### 성능 목표

| 지표 | 목표 |
|---|---|
| 교차 진단 응답 p95 | ≤ 3초 |
| 교집합 0곳 시 대안 제안 | ≥ 2개 |
| 수도권 커버리지 | ≥ 90% |

## 근거

- **AOS 4.00** (중요도 5.0, 만족도 1.0) — 20개 Pain Point 중 공동 1위 ([[src-aos-dos]])
- **DOS 4.00** — MR 1.0으로 AOS와 동일. 서비스 정체성 그 자체 ([[src-aos-dos]])
- 폐업 사유 1위가 입지 선정 실패(25%) — 이 기능이 문제의 핵심을 해결 ([[src-market-analysis]])
- 기존 시장에 두 동선 동시 계산 도구가 전무 ([[src-problem-definition]])

## MVP 구현 상태

- ✅ **MVP 포함** (Phase 2, Day 9-18)
- REQ-FUNC-001~008

## UI 컴포넌트

| 컴포넌트 | 라이브러리 | 기능 |
|---|---|---|
| `DiagnosisMap` | react-kakao-maps-sdk | 후보 동네 마커 + 폴리곤 |
| `CandidateCard` | shadcn/ui Card | 통근·가격·안전 요약 |
| `FilterPanel` | shadcn/ui Slider | 통근시간·예산 필터 |
| `AddressInput` | shadcn/ui Combobox | 카카오 Geocoding 자동완성 |

## 관련 페이지

- Entities: [[diagnosis]], [[user]]
- Sources: [[src-srs]], [[src-prd]], [[src-aos-dos]], [[src-implementation-plan]]
- Concepts: [[deadline-mode]], [[share-link]], [[tech-stack]]
