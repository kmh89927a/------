---
title: "Wiki 이력 로그"
category: meta
created: 2026-04-23
updated: 2026-04-23
---

# Wiki 이력 로그 (Log)

> 시간순 기록. 각 항목은 `## [날짜] 작업유형 | 설명` 형식.

---

## [2026-04-23] ingest | 초기 지식베이스 구축 (Batch Ingest — 15개 원본 문서)

### 작업 내용

`raw/assets/` 디렉토리의 15개 원본 문서를 분석하여 Wiki 지식베이스의 초기 구조를 구축했습니다.

### 생성된 파일 목록

**Schema & Meta (3개)**
- `_schema.md` — Wiki 운영 규칙
- `index.md` — 콘텐츠 카탈로그
- `overview.md` — 프로젝트 전체 개요

**Sources (15개)**
- `sources/src-prd.md` ← `00_PRD_v1.1-rev.4.md`
- `sources/src-srs.md` ← `05_SRS_v1.6.md`
- `sources/src-task-list.md` ← `06_TASK_LIST_v1.3.md`
- `sources/src-implementation-plan.md` ← `plan_srs_stack_alignment_implementation_v1.2.md`
- `sources/src-value-proposition.md` ← `06_value-proposition-sheet_V2_(Rooted).md`
- `sources/src-jtbd.md` ← `10_jtbd-interview-report.md`
- `sources/src-market-analysis.md` ← `6_TAM-SAM-SOM+MarketSegment.md`
- `sources/src-persona.md` ← `7_persona-spectrum-map.md`
- `sources/src-cjm.md` ← `8_customer-journey-map.md`
- `sources/src-aos-dos.md` ← `9_aos-dos-analysis.md`
- `sources/src-problem-definition.md` ← `5_problem-definition.md`
- `sources/src-competitor-analysis.md` ← `2_competents-analysis.md`
- `sources/src-ksf.md` ← `4_ksf-report.md`
- `sources/src-porters-forces.md` ← `1_porters-foreces.md` ⚠️ 원본 미완성
- `sources/src-value-chain.md` ← `3_value-chain.md` ⚠️ 원본 미완성

**Entities (5개)**
- `entities/user.md` — User 엔터티
- `entities/diagnosis.md` — Diagnosis 엔터티
- `entities/share-link-entity.md` — ShareLink 엔터티
- `entities/saved-search.md` — SavedSearch 엔터티
- `entities/persona-spectrum.md` — 12명 페르소나

**Concepts (6개)**
- `concepts/two-route-intersection.md` — F1 두 동선 교차 진단
- `concepts/share-link.md` — F2 배우자 공유 링크
- `concepts/deadline-mode.md` — F3 데드라인 모드
- `concepts/single-mode.md` — F4 싱글 모드
- `concepts/tech-stack.md` — 기술 스택
- `concepts/market-size.md` — 시장 규모

### 발견된 이슈

1. `1_porters-foreces.md` — 원본에 문제정의서 초안이 잘못 덮어씌워져 있음. → ✅ 폐기 처리 (2026-04-23)
2. `3_value-chain.md` — 원본에 KSF 내용이 잘못 기입됨. → ✅ 폐기 처리 (2026-04-23)

### 다음 단계 (추천)

- [x] ~~Porter's Five Forces 원본 재작성 후 재ingest~~ → 폐기, `src-problem-definition.md`에 통합
- [x] ~~가치 사슬 분석 원본 재작성 후 재ingest~~ → 폐기, `src-ksf.md`에 통합
- [ ] Lint 패스 실행 — 교차 참조 누락, 고아 페이지, 모순 검사
- [ ] F5 간이 저장 concept 페이지 추가 (`saved-search-concept.md`)

---

## [2026-04-23] cleanup | Porter's 5 Forces, Value Chain 원본 폐기 처리

Porter's 5 Forces(`1_porters-foreces.md`), Value Chain(`3_value-chain.md`) 원본 미완성 파일을 폐기 처리함. 해당 분석 내용은 Problem Definition / KSF 문서에 각각 통합됨을 확인.

### 수행 내용

1. **Wiki 페이지 삭제**: `src-porters-forces.md`, `src-value-chain.md` 제거
2. **카탈로그 정리**: `index.md`에서 미완성 섹션 및 해당 항목 제거, Sources 15→13개
3. **통합 안내 추가**: Porter's 분석 → `src-problem-definition.md`, Value Chain → `src-ksf.md`에 통합 블록 추가
4. **원본 아카이브**: `raw/_archive/`로 이동 (DEPRECATED 접미사)

### 영향 받은 파일

- ❌ 삭제: `wiki/sources/src-porters-forces.md`, `wiki/sources/src-value-chain.md`
- ✏️ 수정: `wiki/index.md`, `wiki/sources/src-problem-definition.md`, `wiki/sources/src-ksf.md`, `wiki/log.md`
- 📦 아카이브: `raw/_archive/1_porters-foreces_DEPRECATED.md`, `raw/_archive/3_value-chain_DEPRECATED.md`

