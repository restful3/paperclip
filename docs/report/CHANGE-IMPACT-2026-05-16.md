# Change-Impact Report — 2026-05-16

**대상 범위**: `b947a7d7` 이후 origin/master 16개 커밋 (`eb452fba` → `4c47eb46`)  
**비교 대상**: 학습 리포트 (`docs/report/00\~10`, `docs/research/01\~10`)  
**직전 기준선**: `b947a7d7` (선행 문서 `CHANGE-IMPACT-2026-05.md` 가 여기까지 커버)  
**작성일**: 2026-05-16  
**판정 기준**: STALE(즉시 수정 필요) / GAP(누락 토픽) / MINOR(소폭 추가) / OK(유지)

---

## 1. 들어오는 변경의 전체 요약

| # | PR | 커밋 | 제목 | 영향 영역 |
|---|---|---|---|---|
| 1 | #5919 | `eb452fba` | Fix comment date binding regression | server |
| 2 | #5938 | `012a7387` | Add ordered sub-issue navigation | ui |
| 3 | #5922 | `d1a8c873` | Harden host workspace resumes (remote-sandbox) | adapters, server |
| 4 | #5603 | `4142559c` | Add blocked inbox attention view | shared, server, ui |
| 5 | #5944 | `f4bed4a7` | Release changelog v2026.513.0 | releases |
| 6 | #5967 | `1bd44c8a` | Harden Cloudflare sandbox execution | adapters, server |
| 7 | #5960 | `333a16b0` | Fix company export with missing run logs | server |
| 8 | #6026 | `901c088e` | Propagate projectId into wakeup context | server |
| 9 | #6009 | `03ad5c5b` | Add issue document locking | db, server, ui, shared, plugins |
| 10 | #6005 | `dfcebf08` | Refresh issue documents from live updates | ui |
| 11 | #6010 | `eb38b226` | Fix LLM Wiki package and migration validation | plugins, server |
| 12 | #6058 | `d5ba3348` | Add UI i18n runtime packages | ui |
| 13 | #6062 | `7e1a27c8` | Refresh pnpm-lock.yaml | lockfile |
| 14 | #5309 | `afb73ba5` | Scale issue kanban board for high-volume columns | ui |
| 15 | #5943 | `e2d7263b` | Add minimal i18next i18n foundation | ui |
| 16 | #6069 | `4c47eb46` | Add multilingual issue preservation coverage | adapter-utils, ui |

**규모**: 약 8,800 insertions / 약 430 deletions (16 커밋 합산).

**검증된 핵심 변경**:

- `packages/db/src/migrations/0085_tranquil_the_executioner.sql` — 신규 마이그레이션 (문서 락 컬럼)
- `packages/db/src/schema/documents.ts` — `locked_at` · `locked_by_agent_id` · `locked_by_user_id` 3개 컬럼 추가
- `server/src/services/documents.ts` — +267 LOC (문서 락/언락 서비스)
- `server/src/services/issues.ts` — +979 LOC (blocked inbox, #5603) → checkout 코드가 약 +995행 하향 이동
- `ui/src/i18n/` — 신규 디렉터리 (i18next foundation, #5943)
- `releases/v2026.513.0.md` — 릴리스 노트

---

## 2. 챕터별 판정

### Tier 1 — STALE (즉시 수정 필요)

#### `02-data-model` — STALE

- §5 atomic checkout 의사코드의 실제 코드 인용 `server/src/services/issues.ts:3593-3736` 가 #5603(`4142559c`) 의 +979 LOC 삽입으로 하향 이동했다. 검증: baseline `b947a7d7` 에서 `checkout: async (...)` 가 3593행, 현재 4588행 → 일괄 +995행. **수정값: `4588-4731`**.
- 부수: `documents` 테이블에 락 컬럼 3개 추가(#6009). 본문은 `issue_documents` 를 이름으로만 나열하므로 컬럼 단위 STALE 아님. `pgTable` 86개 / 스키마 파일 78개 / `index.ts:1-78` 카운트는 신규 테이블이 없어 그대로 유효(재검증 완료).

#### `03-runtime-execution` — STALE

- §1 그림 3-1 본문의 atomic checkout 코드 인용 `server/src/services/issues.ts:3593-3736` 동일 이슈. **수정값: `4588-4731`**.
- 부수(MINOR): #6026(`901c088e`) 가 wakeup 컨텍스트에 `projectId` 전파 + identifier 조회를 추가. heartbeat 컨텍스트 필드 설명에 한 줄 보강 여지(GAP-경계).

#### `05-server-api` — STALE

- §2 표 1 및 본문의 `routes/issues.ts` 크기 `168 KB` 표기가 실제와 불일치. #5603(+49행) + #6009(+94행) 으로 `routes/issues.ts` 가 184,127바이트(약 180 KB)로 증가. baseline 에서도 이미 174 KB 였으므로 표기는 선행 누락분 포함. **수정값: `180 KB`** (3개소: §2 표 1 행, §2 본문, §3 본문).

#### `09-pros-cons` — STALE

- "단일 assignee + atomic checkout" 행의 코드 인용 `server/src/services/issues.ts:3593-3736`. **수정값: `4588-4731`**.
- "거대 단일 파일들" 행의 `routes/issues.ts 168 KB`. **수정값: `180 KB`**.

### Tier 2 — GAP (누락 토픽, 사용자 확인 후 별도 반영)

#### `06-ui-and-board` — GAP (다수)

직전 기준선 이후 UI에 신규 기능이 다수 추가됐다. 본문에 거짓 진술은 없으나 토픽 누락:

- **i18n 기반(#5943 · #6058 · #6069)** — `ui/src/i18n/` 신규(i18next + locale validation). 리포트 전체에 i18n 언급 0건. 다만 "minimal foundation" 단계.
- **blocked inbox attention view(#5603)** — shared 타입 + server 서비스(+979 LOC) + UI. 규모가 큰 신규 기능.
- **kanban 보드 고용량 컬럼 스케일링(#5309)** — UI 성능 기능(+904 LOC).
- **ordered sub-issue navigation(#5938)** — 하위 이슈 순서 탐색(+763 LOC).
- **issue document 라이브 갱신(#6005)** — 문서 실시간 새로고침.

#### `07-governance-cost` — GAP

- **issue document locking(#6009)** 은 보드 운영자가 issue document 를 잠그고/푸는 신규 거버넌스 레버다(잠긴 문서는 불변, 에이전트 쓰기는 derived key 생성). 본문 보드 권한 열거(승인·예산·일시정지)에 누락. `POST /issues/:id/documents/:key/lock` · `/unlock` 2개 라우트도 05장 GAP.

### Tier 3 — MINOR / OK

| 챕터 | 판정 | 사유 |
|---|---|---|
| `04-adapters-and-skills` | MINOR | Cloudflare sandbox 강화(#5967) · host workspace resume 강화(#5922) · LLM Wiki 수정(#6010) — 모두 기존 커버 항목의 hardening/fix |
| `01-architecture` | MINOR | `ui/src/i18n/` 신규 디렉터리. 패키지 지도는 coarse 단위라 즉시 수정 불요 |
| `00-overview` `08-usage` `10-research-map` | OK | 영향 없음 |
| `docs/research/01\~10` | OK | 외부 기술 분석 문서 — 16커밋의 영향 범위 밖 (i18next 는 research/08 에 한 줄 추가 여지 있으나 MINOR 미만) |

버그 수정 커밋(#5919 comment date · #5960 export run logs · #6010 LLM Wiki) 과 릴리스/락파일(#5944 · #6062 · #6058) 은 아키텍처 서술을 바꾸지 않아 본문 OK.

---

## 3. 판정 카운트

`docs/report/` 11개 + `docs/research/` 10개 = 총 **21개** 분석 대상.

| 판정 | 개수 | 챕터 |
|---|---|---|
| STALE | 4 | `02-data-model`, `03-runtime-execution`, `05-server-api`, `09-pros-cons` |
| GAP | 2 | `06-ui-and-board`, `07-governance-cost` |
| MINOR | 2 | `01-architecture`, `04-adapters-and-skills` |
| OK | 13 | `00-overview`, `08-usage`, `10-research-map`, `docs/research/01\~10` |
| **합계** | **21** | |

---

## 4. 적용 결과 (STALE 4개 — 2026-05-16 반영 완료)

`/loop` 사이클 Step 4 에서 STALE 판정분만 본문 반영:

- `02-data-model.md` §5 — `issues.ts:3593-3736` → `issues.ts:4588-4731`
- `03-runtime-execution.md` §1 — `issues.ts:3593-3736` → `issues.ts:4588-4731`
- `09-pros-cons.md` — `issues.ts:3593-3736` → `issues.ts:4588-4731`, `routes/issues.ts 168 KB` → `180 KB`
- `05-server-api.md` §2·§3 — `issues.ts 168 KB` → `180 KB` (3개소)
- `_pdf-build/` 챕터 사본 재동기화 + PDF/HTML 재빌드

**GAP 2개(06 · 07)** 와 MINOR 는 본 사이클 범위 밖 — 사용자 확인 후 별도 반영 대상.

---

## 5. 메모

- 모든 LOC·라인번호·파일 크기는 2026-05-16 시점 `4c47eb46`(origin/master) 기준으로 직접 검증함.
- checkout 코드 이동량 +995행은 def 행(3593→4588)과 `inArray(issues.status, ...)` 행(3648→4643) 두 지점에서 동일하게 확인 — 블록 내부는 16커밋이 건드리지 않아 균일 이동.
- `server/src/adapters/registry.ts` 라인 인용(00·01·04장)은 16커밋이 해당 파일을 건드리지 않아 STALE 아님.
- 본 사이클은 origin/master 16커밋을 로컬 master 에 rebase 동기화한 직후 작성.
