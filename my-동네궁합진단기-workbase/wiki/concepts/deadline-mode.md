---
title: "데드라인 모드 (F3)"
category: concepts
tags: [mvp, 기술]
sources: [src-srs, src-prd, src-aos-dos]
created: 2026-04-23
updated: 2026-04-23
status: active
---

# 데드라인 모드 (Deadline Mode)

## 정의

이사 마감일을 입력하면 계약 역산 타임라인(5단계+)을 생성하고, 교집합 동네의 매물을 네이버 부동산 아웃링크로 연결하는 긴급 이사자 전용 기능(F3).

## 상세

### 타임라인 생성

- Server Action에서 D-day 기반 자동 산출 (≥ 5단계)
- 과거 날짜 입력 시 클라이언트 검증으로 100% 차단 (서버 도달률 0%)

### 매물 조회 — 네이버 부동산 아웃링크

| 항목 | v1.1 (크롤링) | v1.2 (아웃링크) |
|---|---|---|
| 데이터 소스 | 직방·피터팬 크롤링 | **네이버 부동산 URL** |
| 인프라 | Vercel Cron Job | **없음** |
| DB 모델 | Listing 테이블 | **불필요** |
| 법적 리스크 | robots.txt 위반 가능 | **없음** |

### 30분 요약 카드

- Top 3 동네 핵심정보 (통근·가격·안전)
- 항목 ≥ 6개/카드

## 근거

- **AOS 3.80, DOS 2.66 (2위)** ([[src-aos-dos]])
- ④정우진 "2달 안에 못 구하면 비상" — 극도 조급 감정 ([[src-cjm]])
- 전체 이사 수요의 ~30%가 비자발적 긴급 이사 ([[src-market-analysis]])

## MVP 구현 상태

- ✅ **MVP 포함** (Phase 4, Day 25-30)
- REQ-FUNC-015~020

## 관련 페이지

- Entities: [[diagnosis]]
- Sources: [[src-srs]], [[src-prd]], [[src-aos-dos]], [[src-cjm]]
- Concepts: [[two-route-intersection]]
