---
title: "배우자 공유 링크 (F2)"
category: concepts
tags: [mvp, 비즈니스]
sources: [src-srs, src-prd, src-aos-dos]
created: 2026-04-23
updated: 2026-04-23
status: active
---

# 배우자 공유 링크 (Share Link)

## 정의

진단 결과를 배우자(비회원)에게 카카오톡/URL로 공유하여, 앱 설치 없이 리포트를 열람하게 하는 기능(F2). 서비스의 바이럴 루프이자 유료 전환 퍼널의 핵심.

## 상세

### 플로우

1. 사용자가 "공유 링크 생성" 클릭
2. Server Action: UUID v4 생성 (entropy ≥ 128bit) + ShareLink DB 저장
3. 클립보드에 URL 복사 → 카카오톡으로 전송
4. 배우자가 링크 클릭 → SSR 공유 페이지 (비회원, 앱 설치 불요)
5. 무료 미리보기 1곳 → 2곳째 접근 시 WTP 설문 유도 모달

### 핵심 기능

| 기능 | 설명 | REQ |
|---|---|---|
| UUID v4 토큰 | entropy ≥ 128bit | FUNC-009 |
| 30일 만료 | 만료 후 개인정보 노출 0건 | FUNC-010 |
| 비회원 열람 | SSR 페이지, 앱 설치 불요 | FUNC-011 |
| 데이터 출처 배지 | 모든 수치에 소스·갱신일 표시 | FUNC-012 |
| 무료 미리보기 1곳 | freePreviewUsed 플래그 | FUNC-013 |
| WTP 설문 모달 | 2곳째 접근 시 ≤ 300ms | FUNC-014 |
| OG 리치 프리뷰 | generateMetadata() | 카카오톡 미리보기 |

## 근거

- **AOS 3.04, DOS 2.52 (3위)** — AOS 6위에서 DOS 3위로 상승. 구현 단순 + 결제 전환·바이럴 직결 ([[src-aos-dos]])
- 의사결정 단위가 **부부**라서, 한 사람의 납득만으로는 이사가 진행되지 않음 ([[src-jtbd]])
- "남편 설득용 데이터 리포트"가 ①김지영의 최우선 니즈 ([[src-cjm]])
- KSF 3번: 배우자 설득 도구 ([[src-ksf]])

## MVP 구현 상태

- ✅ **MVP 포함** (Phase 3, Day 19-24)
- REQ-FUNC-009~014

## 관련 페이지

- Entities: [[share-link-entity]], [[diagnosis]]
- Sources: [[src-srs]], [[src-prd]], [[src-aos-dos]], [[src-cjm]]
- Concepts: [[two-route-intersection]]
