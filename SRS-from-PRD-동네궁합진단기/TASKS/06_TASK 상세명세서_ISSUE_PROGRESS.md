# TASK 상세 명세서 (GitHub Issue) 작성 진행 현황

**최종 갱신:** 2026-04-18T15:28:00+09:00
**Source:** TASK_LIST_v1.md (94개 태스크)

---

## 현재 작업 단위: Wave 1 — 기반 구축 (Critical Path)

### 작성 완료된 상세 명세서 (10/94)

| # | 파일 | Task ID | Epic | 핵심 내용 | 복잡도 | 크기 |
|---|---|---|---|---|---|---|
| 1 | [ISSUE_INFRA-001.md](./ISSUE_INFRA-001.md) | **INFRA-001** | Infra | Next.js 15+ App Router 프로젝트 초기화 + Vercel 배포 파이프라인 (Git Push 자동 배포) | M | 6.7KB |
| 2 | [ISSUE_DB-001.md](./ISSUE_DB-001.md) | **DB-001** | Data Foundation | Prisma ORM 초기화 + SQLite/Supabase 환경변수 기반 전환 구조 + 싱글톤 클라이언트 | L | 7.0KB |
| 3 | [ISSUE_DB-002.md](./ISSUE_DB-002.md) | **DB-002** | Data Foundation | USER 테이블 Prisma 스키마 (AuthProvider·ServiceMode enum, email UNIQUE) | L | 7.0KB |
| 4 | [ISSUE_DB-003.md](./ISSUE_DB-003.md) | **DB-003** | Data Foundation | DIAGNOSIS 테이블 Prisma 스키마 (JSONB filters, FK→User, 복합 인덱스, CASCADE) | M | 9.2KB |
| 5 | [ISSUE_API-006.md](./ISSUE_API-006.md) | **API-006** | API Contract | 공통 에러 코드 체계 — 6개 도메인 20+ 코드, 3-layer 매핑 (HTTP·앱코드·메시지) | M | 11.6KB |
| 6 | [ISSUE_DB-004.md](./ISSUE_DB-004.md) | **DB-004** | Data Foundation | SHARE_LINK 테이블 Prisma 스키마 (UUID v4 고유 URL, 30일 만료, 비밀번호 보호, 미리보기 추적) | L | 5.8KB |
| 7 | [ISSUE_DB-005.md](./ISSUE_DB-005.md) | **DB-005** | Data Foundation | PAYMENT 테이블 Prisma 스키마 (1회/구독 플랜, PG 트랜잭션 추적, 4단계 상태 머신) | M | 7.2KB |
| 8 | [ISSUE_DB-007.md](./ISSUE_DB-007.md) | **DB-007** | Data Foundation | NextAuth.js Prisma Adapter (Account·Session·VerificationToken) + User 모델 확장 | M | 8.5KB |
| 9 | [ISSUE_API-007.md](./ISSUE_API-007.md) | **API-007** | API Contract | 카카오 모빌리티 API 타입 체계 (경로 요청/응답·에러·클라이언트 인터페이스·후보 동네 타입) | L | 9.1KB |
| 10 | [ISSUE_API-008.md](./ISSUE_API-008.md) | **API-008** | API Contract | 토스페이먼츠 PG 연동 타입 (결제 요청/승인/웹훅/서명 검증·PCI-DSS 준수) | M | 10.2KB |

### 각 Issue에 포함된 섹션 (Template 준수 확인)

| 섹션 | 포함 여부 | 설명 |
|---|---|---|
| :dart: Summary | ✅ | 기능명·목적·SRS 근거 |
| :link: References | ✅ | SRS 파일 내 정확한 섹션 링크 (ERD, 시퀀스, CLD 등) |
| :white_check_mark: Task Breakdown | ✅ | 실행 가능한 체크리스트 + 코드 스니펫 포함 |
| :test_tube: Acceptance Criteria (GWT) | ✅ | 4~6개 BDD 시나리오 (Given/When/Then) |
| :gear: Technical Constraints | ✅ | 성능·보안·호환성·비용 제약 |
| :checkered_flag: Definition of Done | ✅ | 완료 판단 체크리스트 |
| :construction: Dependencies & Blockers | ✅ | Depends on / Blocks 명시 |

---

## 작성 이력

### Batch 1 (5건) — 2026-04-18T15:10
- INFRA-001, DB-001, DB-002, DB-003, API-006
- 선정 기준: Critical Path 최선두 + 외부 의존 없는 병렬 태스크

### Batch 2 (5건) — 2026-04-18T15:28
- DB-004, DB-005, DB-007, API-007, API-008
- 선정 기준: Wave 1 나머지 — DB 스키마 완성 + 외부 API 계약 확립

```
                              ┌── DB-004 (SHARE_LINK) ✅
              ┌── DB-003 ✅ ──┤
DB-001 ✅ → DB-002 ✅ ──┤     └── (DB-003 하위 완료)
              │          ├── DB-005 (PAYMENT) ✅
              │          └── DB-007 (NextAuth) ✅
              └── DB-006, DB-008~010 (미작성)

API-006 ✅ (병렬)    API-007 ✅ (병렬)    API-008 ✅ (병렬)
```

---

## 미작성 태스크 현황 (84/94 잔여)

### 다음 작업 단위 후보 (Wave 1 잔여 + Wave 2)

| 우선순위 | Task ID | Feature | 사유 |
|---|---|---|---|
| 1 | **DB-006** | SavedSearch 테이블 스키마 | DB-002 완료 상태, SAVE 도메인 선행 |
| 2 | **DB-008** | 행정동 코드 매핑 Seed 데이터 | DB-001 완료 상태, 재탐색 행정동 변경 감지용 |
| 3 | **DB-009** | 경찰청 범죄 통계 캐시 Seed 데이터 | DB-001 완료 상태, 싱글 모드 야간 치안 등급용 |
| 4 | **DB-010** | 교육부 학교 배정 구역 Seed 데이터 | DB-001 완료 상태, 학교 배정 오버레이용 |
| 5 | **API-001** | Auth 도메인 DTO | DB-007 완료 상태, NextAuth 세션·콜백 타입 |

### 카테고리별 잔여 현황

| 카테고리 | 전체 | 완료 | 잔여 |
|---|---|---|---|
| Step 1: DB 스키마 | 10 | **6** | **4** |
| Step 1: API Contract | 8 | **3** | **5** |
| Step 1: Mock | 5 | 0 | **5** |
| Step 2: Command/Query | 34 | 0 | **34** |
| Step 3: Test | 10 | 0 | **10** |
| Step 4: Infra/Sec/Mon | 12 | 1 | **11** |
| Step 5: UI/UX | 15 | 0 | **15** |
| **합계** | **94** | **10** | **84** |

### 진행률 시각화

```
전체 진행률: [██░░░░░░░░░░░░░░░░░░] 10.6% (10/94)

Step 1 — DB 스키마:    [████████████░░░░░░░░] 60%  (6/10)
Step 1 — API Contract: [████████░░░░░░░░░░░░] 37%  (3/8)
Step 1 — Mock:         [░░░░░░░░░░░░░░░░░░░░]  0%  (0/5)
Step 2 — Cmd/Query:    [░░░░░░░░░░░░░░░░░░░░]  0%  (0/34)
Step 3 — Test:         [░░░░░░░░░░░░░░░░░░░░]  0%  (0/10)
Step 4 — Infra/NFR:    [██░░░░░░░░░░░░░░░░░░]  8%  (1/12)
Step 5 — UI/UX:        [░░░░░░░░░░░░░░░░░░░░]  0%  (0/15)
```

---

> 💡 **운영 가이드:** 새로운 ISSUE 파일 작성 시 이 문서의 "작성 완료된 상세 명세서" 테이블에 행을 추가하고, "카테고리별 잔여 현황"을 갱신할 것.
