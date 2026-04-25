---
title: "싱글 모드 (F4)"
category: concepts
tags: [mvp, 기술]
sources: [src-srs, src-prd]
created: 2026-04-23
updated: 2026-04-23
status: active
---

# 싱글 모드 (Single Mode)

## 정의

자녀 없는 싱글 사용자(이직 후 이사 등)를 위해 학군·가족 항목을 자동 숨기고, 야간 치안 등급·편의시설·카페 밀집도를 강조하는 간소화 진단 모드(F4).

## 상세

### 모드 분기

- `mode === 'single'` 조건부 렌더링
- 학군·가족 항목: 0건 표시 (자동 숨김)
- 야간 치안 등급(A~D): 경찰청 범죄 통계 기반 (22~06시 범죄 건수)
- 편의시설·카페 밀집도 레이어: 기본 활성화

### 리포트 저장

| 항목 | v1.1 | v1.2 |
|---|---|---|
| 렌더링 | @react-pdf/renderer 서버 | **window.print()** |
| 의존성 | ~5MB 라이브러리 | **브라우저 기본** |
| 응답 시간 | ≤ 3초 | **≤ 1초** |
| 서버 비용 | CPU 부하 | **0원** |

### 비수도권 차단

- Client validation ≤ 500ms + 지원 지역 목록 표시
- 커버리지: 서울/경기/인천 (수도권 90%)

## 근거

- ⑥이준혁(이직 후 이사) Adjacent 페르소나의 "직장+여가 2거점" 니즈 ([[src-persona]])
- "싱글인데 학군 항목이 있으면 불필요" 피드백 ([[src-cjm]])

## MVP 구현 상태

- ✅ **MVP 포함** (Phase 5, Day 31-35)
- REQ-FUNC-021~024

## 관련 페이지

- Entities: [[diagnosis]], [[user]]
- Sources: [[src-srs]], [[src-prd]], [[src-persona]]
- Concepts: [[two-route-intersection]]
