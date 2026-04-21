# SRS Rev 1.6 자기 검증 체크리스트

## A. Auth 스택 전환 (NextAuth.js v5 → Supabase Auth)

| # | 검증 항목 | 결과 | 비고 |
| --- | --- | --- | --- |
| A-1 | CON-10에서 NextAuth.js 언급 제거 | ✅ | CON-10에 원래 NextAuth 언급 없었음 (Server Actions/Route Handlers만) |
| A-2 | CON-18 신규 추가 (Supabase Auth) | ✅ | Line 162 |
| A-3 | §3.3 API Overview 설명문 Supabase Auth 변경 | ✅ | Line 273 |
| A-4 | API 인증 열 NextAuth Session → Supabase Session | ✅ | §3.3 + §6.1 모두 변경 |
| A-5 | API-07 엔드포인트 `/auth/callback` 변경 | ✅ | Line 282, 637 |
| A-6 | API-08 제거 | ✅ | §3.3, §6.1 모두 제거 확인 |
| A-7 | REQ-FUNC-029 Supabase Auth 재작성 | ✅ | Line 477 |
| A-8 | REQ-NF-018 Supabase Auth httpOnly cookie | ✅ | Line 525, maxAge/updateAge 수치 제거 |
| A-9 | §6.3.6 시퀀스 Supabase Auth 재작성 | ✅ | Lines 1015-1057 |
| A-10 | §6.6 AuthMod Supabase Auth 교체 | ✅ | Line 1170 |
| A-11 | NextAuth 검색 시 잔존 여부 | ⚠️ | 3건 잔존: Changelog 제목(L59), CON-18 부정문(L162), Rev 1.3 기록(L1353) — 모두 히스토리 기록 또는 명시적 미사용 선언이므로 정상 |

## B. 결제 MVP 제외

| # | 검증 항목 | 결과 | 비고 |
| --- | --- | --- | --- |
| B-1 | IS-07 제거 | ✅ | In-Scope 테이블에서 제거, Changelog에만 기록 |
| B-2 | OS-09 신규 추가 | ✅ | Line 139 |
| B-3 | EXT-06 토스페이먼츠 제거 + 우회 전략 제거 | ✅ | §3.1 양쪽 테이블에서 제거, 토스페이먼츠 검색 시 changelog에만 1건 |
| B-4 | API-09, API-10 제거 | ✅ | §3.3, §6.1 모두 제거 |
| B-5 | §4.1.6 제목 "인증·결제" → "인증" | ✅ | |
| B-6 | REQ-FUNC-030 제거 | ✅ | 본문에서 제거, Changelog/Footer에만 기록 |
| B-7 | REQ-NF-025 제거 | ✅ | |
| B-8 | REQ-NF-026 "진단 완료 수"로 변경 | ✅ | |
| B-9 | REQ-NF-027 "WTP 설문 응답률"로 변경 | ✅ | |
| B-10 | REQ-NF-032 측정 방법 "설문으로 측정" 추가 | ✅ | |
| B-11 | §5.1 트레이서빌리티 "인증" + 030 제거 | ✅ | |
| B-12 | §5.3 KPI 매핑 REQ-FUNC-030 제거 | ✅ | |
| B-13 | §6.2.0 ERD PAYMENT 엔터티·관계 제거 | ✅ | |
| B-14 | §6.2.4 PAYMENT 섹션 제거 | ✅ | |
| B-15 | §6.3.2 PG 참가자·결제 하위 플로우 제거 | ✅ | |
| B-16 | §6.3.6 결제 플로우 제거, 제목 "인증 플로우" | ✅ | |
| B-17 | §6.5 UC-06 제거 + Actor 화살표 제거 | ✅ | |
| B-18 | §6.6 PayMod·TossPG 제거 + 연결선 제거 | ✅ | |
| B-19 | §6.7 Payment 클래스·Enum·관계 제거 | ✅ | |
| B-20 | PAYMENT 검색 시 잔존 | ✅ | Changelog/Footer 기록에만 2건 — 정상 |

## C. 버전 참조

| # | 검증 항목 | 결과 | 비고 |
| --- | --- | --- | --- |
| C-1 | 메타 표 Source PRD v0.1-rev.4 | ✅ | Line 12 |
| C-2 | REF-01 버전 v0.1-rev.4 | ✅ | Line 201 |
| C-3 | Footer Rev 1.6 | ✅ | Line 1347 |
| C-4 | Footer PRD v0.1-rev.4 | ✅ | Line 1349 |

## D. 잔존 정합성 오류 수선

| # | 검증 항목 | 결과 | 비고 |
| --- | --- | --- | --- |
| D-1 | REQ-NF-009 공백 행 추가 | ✅ | Line 510 |
| D-2 | REQ-NF-016 들여쓰기·셀 정렬 수정 | ✅ | 공백 행 제거, REQ-NF-013 바로 다음에 배치 |

## E. 구조 보존 확인

| # | 검증 항목 | 결과 |
| --- | --- | --- |
| E-1 | 기존 ID 재사용 없음 | ✅ |
| E-2 | 제거된 ID 공백 유지 (REQ-NF-009) | ✅ |
| E-3 | Changelog 블록 갱신 | ✅ |
| E-4 | 표·Mermaid 구조 보존 (행/노드만 가감) | ✅ |
| E-5 | Rev 1.6 Changelog 삽입 | ✅ |

## [QUESTION]

해당 작업에서 지시에 없는 추가 판단이 필요한 사항은 없었습니다.
