---
name: Feature Task
about: SRS 기반의 구체적인 개발 태스크 명세
title: "[DB] DB-004: SHARE_LINK 테이블 Prisma 스키마 정의 및 마이그레이션"
labels: 'database, backend, priority:high'
assignees: ''
---

## :dart: Summary
- **기능명:** [DB-004] SHARE_LINK 테이블 Prisma 스키마 정의 및 마이그레이션
- **목적:** SRS §6.2.3에 정의된 SHARE_LINK 엔터티를 Prisma 스키마로 구현한다. SHARE_LINK는 배우자 공유 링크(F2)의 핵심 데이터 모델로, UUID v4 기반 고유 URL 생성, 30일 만료 관리, 선택적 비밀번호 보호, 무료 미리보기 1곳 추적, 열람 횟수 카운팅을 지원한다. DIAGNOSIS 테이블을 FK로 참조하며, 공유 링크를 통한 비회원 리포트 열람의 전체 라이프사이클을 관리한다.

## :link: References (Spec & Context)
> :bulb: AI Agent & Dev Note: 작업 시작 전 아래 문서를 반드시 먼저 Read/Evaluate 할 것.
- SRS SHARE_LINK 데이터 모델: [`/TASKS/SRS_v1.md#6.2.3 SHARE_LINK`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — 8개 필드 전체 명세
- SRS ERD: [`/TASKS/SRS_v1.md#6.2.0 ERD`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — DIAGNOSIS(1:N) → SHARE_LINK
- SRS 공유 링크 시퀀스: [`/TASKS/SRS_v1.md#6.3.2`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — createShareLink → Prisma ShareLink create
- SRS 공유 링크 요구사항: [`/TASKS/SRS_v1.md#4.1.2`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — REQ-FUNC-009~014
- SRS 보안: [`/TASKS/SRS_v1.md#4.2.3`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md) — REQ-NF-020 (URL entropy ≥ 128bit), REQ-NF-021 (비인가 접근 차단)
- SRS API 명세: [`/TASKS/SRS_v1.md#6.1 API-03/04`](file:///Users/mihee/기획개발강의/SRS-from-PRD-동네궁합진단기/TASKS/SRS_v1.md)

## :white_check_mark: Task Breakdown (실행 계획)
- [ ] **T1.** `prisma/schema.prisma`에 ShareLink 모델 정의:
  ```prisma
  model ShareLink {
    id              String    @id @default(uuid())
    diagnosisId     String    @map("diagnosis_id")
    uniqueUrl       String    @unique @map("unique_url") @db.VarChar(255)
    passwordHash    String?   @map("password_hash") @db.VarChar(255)
    viewCount       Int       @default(0) @map("view_count")
    freePreviewUsed Boolean   @default(false) @map("free_preview_used")
    expiresAt       DateTime  @map("expires_at") @db.Date
    createdAt       DateTime  @default(now()) @map("created_at")

    // Relations
    diagnosis       Diagnosis @relation(fields: [diagnosisId], references: [id], onDelete: Cascade)

    // Indexes
    @@index([diagnosisId])
    @@index([expiresAt])
    @@map("share_links")
  }
  ```
- [ ] **T2.** Diagnosis 모델에 `shareLinks ShareLink[]` relation 추가 확인
- [ ] **T3.** 인덱스 설정:
  - `uniqueUrl`: UNIQUE 인덱스 (스키마 @unique)
  - `diagnosisId`: 진단별 공유 링크 조회
  - `expiresAt`: 만료 링크 정리(Cleanup) 배치 쿼리 최적화
- [ ] **T4.** 마이그레이션 실행: `npx prisma migrate dev --name add-share-link-table`
- [ ] **T5.** 마이그레이션 SQL 검증 — SRS §6.2.3 대비 8개 필드 1:1 매핑 확인
- [ ] **T6.** Prisma Studio에서 ShareLink + Diagnosis 관계 CRUD 검증

## :test_tube: Acceptance Criteria (BDD/GWT)

**Scenario 1: 정상적인 ShareLink 생성**
- **Given:** Diagnosis 레코드가 존재함
- **When:** `prisma.shareLink.create({ data: { diagnosisId, uniqueUrl: uuid(), expiresAt: now()+30d } })`를 실행함
- **Then:** share_links 테이블에 레코드 생성. `viewCount`=0, `freePreviewUsed`=false 기본값 설정.

**Scenario 2: uniqueUrl UNIQUE 제약 위반**
- **Given:** `uniqueUrl="abc-123"`인 ShareLink 레코드가 이미 존재
- **When:** 동일한 `uniqueUrl`로 생성 시도
- **Then:** Prisma P2002 에러 발생. 레코드 미생성.

**Scenario 3: viewCount 증가**
- **Given:** `viewCount=0`인 ShareLink 레코드 존재
- **When:** `prisma.shareLink.update({ data: { viewCount: { increment: 1 } } })`를 실행
- **Then:** `viewCount`가 1로 증가.

**Scenario 4: CASCADE 삭제**
- **Given:** Diagnosis에 연결된 ShareLink 2건 존재
- **When:** 해당 Diagnosis를 삭제
- **Then:** 연결된 ShareLink 2건도 CASCADE 삭제.

**Scenario 5: 비밀번호 해시 nullable 동작**
- **Given:** 빈 share_links 테이블
- **When:** `passwordHash` 없이 ShareLink 생성
- **Then:** `passwordHash`가 null로 저장 (비밀번호 미설정 공유 링크).

## :gear: Technical & Non-Functional Constraints
- **UUID v4 보안 (REQ-NF-020):** `uniqueUrl`은 클라이언트가 아닌 서버에서 `crypto.randomUUID()` 또는 `uuid v4` 라이브러리로 생성. Prisma `@default(uuid())`는 id용이므로, uniqueUrl은 비즈니스 로직에서 생성하여 전달.
- **만료 관리:** `expiresAt`은 `@db.Date` — 시각 없이 날짜만 저장. 만료 판단은 서버 사이드에서 `expiresAt < today()` 비교.
- **비밀번호 해시:** `passwordHash`는 bcrypt 해시 저장 (CMD-SHARE-004에서 구현). 스키마에서는 nullable VARCHAR(255)만 정의.

## :checkered_flag: Definition of Done (DoD)
- [ ] `prisma/schema.prisma`에 ShareLink 모델이 SRS §6.2.3과 1:1 매핑됨
- [ ] `npx prisma migrate dev` 성공
- [ ] `uniqueUrl` UNIQUE 인덱스 확인 (중복 시 P2002)
- [ ] Diagnosis ↔ ShareLink 관계 CASCADE 삭제 동작 확인
- [ ] Prisma Studio CRUD 검증 완료

## :construction: Dependencies & Blockers
- **Depends on:** DB-003 (DIAGNOSIS 테이블 — `diagnosis_id FK → Diagnosis.id`)
- **Blocks:**
  - API-003 (ShareLink 도메인 DTO)
  - CMD-SHARE-001 ~ 004 (ShareLink 도메인 전체 Command/Query)
  - MOCK-002 (공유 링크 Mock 데이터)
