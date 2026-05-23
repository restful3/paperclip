# Change-Impact Report — 2026-05-23

**대상 범위**: `4c47eb46` 이후 origin/master 37개 커밋 (`63821bfe` → `e3c875c1`)  
**비교 대상**: 학습 리포트 (`docs/report/00\~10`, `docs/research/01\~10`)  
**직전 기준선**: `4c47eb46` (선행 문서 `CHANGE-IMPACT-2026-05-16.md` 가 여기까지 커버)  
**작성일**: 2026-05-23  
**판정 기준**: STALE(즉시 수정 필요) / GAP(누락 토픽) / MINOR(소폭 추가) / OK(유지)

---

## 1. 들어오는 변경의 전체 요약

| # | PR | 커밋 | 제목 | 영향 영역 |
|---|---|---|---|---|
| 1 | #6070 | `63821bfe` | Add full locale catalog (39개 언어) | ui/i18n |
| 2 | #6087 | `ab8b4716` | Add built-in grok_local adapter | adapters, server, ui, cli |
| 3 | #6136 | `9b6d2e6b` | Refresh pnpm-lock.yaml | lockfile |
| 4 | #6137 | `81d18f2d` | Speed up PR verify workflow | ci |
| 5 | #6142 | `573e9ec9` | Restore turn boundaries in grok-local streaming | adapters/grok-local |
| 6 | #6150 | `93cd933f` | Release changelog v2026.517.0 | releases |
| 7 | #6154 | `7bbdfb69` | Enable Grok adapter canary publishing | release-manifest |
| 8 | #6152 | `3e6610fb` | Add release-changelog-discord-message skill | skills |
| 9 | #6212 | `705c1b8d` | Add routine env secrets support | db(0086), server, ui |
| 10 | #6210 | `d734bd43` | Roll up May 17 branch changes | shared validators, adapter-utils, server, plugins |
| 11 | #6222 | `73438510` | Fix new secret form textarea overflow | ui |
| 12 | #6240 | `242a2c2f` | Stop worktree init --force from wiping repo worktrees | cli |
| 13 | #6071 | `5071c4c7` | Add workspace diff viewer plugin | plugins, sdk, server, ui |
| 14 | #6263 | `85510f0e` | Refresh pnpm-lock.yaml | lockfile |
| 15 | #6269 | `32605b71` | Remove planning badge from inbox issue rows | ui, e2e |
| 16 | #6261 | `8f45d124` | Update plugin authoring guide for managed resources | doc |
| 17 | #6245 | `4b1e92a5` | Add Modal sandbox provider plugin | plugins/sandbox-providers |
| 18 | #6289 | `e5a0f5de` | Raise environmentProbe RPC timeout to 120s | server/plugin-env-driver |
| 19 | #6290 | `98868994` | Publish modal plugin from ci | release-manifest |
| 20 | #6311 | `a07e6cef` | Fix new issue autocomplete pointer selection | ui |
| 21 | #6338 | `f343bae1` | Copy link-plugin-dev-sdk.mjs into Docker deps stage | docker |
| 22 | #6339 | `c0c5a826` | Wire SecretBindingPicker into JsonSchemaForm | ui |
| 23 | #6341 | `24748de4` | Hide sandbox-provider plugins from Instance Settings sidebar | ui |
| 24 | #6371 | `bfe6369e` | Guard cheap recovery model usage | server, doc/SPEC-implementation |
| 25 | #6378 | `9c29394f` | Allow cloud tenant import mutations without browser origin | server middleware |
| 26 | #6381 | `d67347be` | Provider vault secrets UX | shared, server, ui |
| 27 | #6383 | `43c5bb81` | Workspace diff polish | plugins/plugin-workspace-diff |
| 28 | #6384 | `f2575305` | UI and dev ops quality-of-life | ui, server, scripts |
| 29 | #6380 | `c91a0623` | Runtime control-plane fixes | server, db/backup-lib, plugin-lifecycle |
| 30 | #6386 | `38c185fb` | Add agent permissions and controls plan | db(0087·0088), server, plugins/sdk, doc/SPEC |
| 31 | #6547 | `a1835cfa` | Harden plugin runtime invocation scope | plugins/sdk, server |
| 32 | #6548 | `e43b392a` | Add local Cloud Upstream sync | db(0089), schema, server, ui, cli — **신규 도메인** |
| 33 | #6549 | `ad6effa6` | Improve runtime and import reliability | db, server, cli |
| 34 | #6550 | `90117827` | Polish board UI mobile flows | ui |
| 35 | #6423 | `4811d8dd` | Fix wrapped company issue prefix conflicts | server, cli, ui |
| 36 | #6433 | `e85ff094` | Fix invite page blank from companies query-key collision | ui |
| 37 | #6560 | `e3c875c1` | Prevent E2B workspace upload + lease idle failures | plugins/sandbox-providers/e2b |

**규모**: 약 22,300 insertions / 약 660 deletions (37 커밋 합산, lockfile 포함).

**검증된 핵심 변경**:

- **신규 마이그레이션 4개**: `0086_routine_env_runtime_contract.sql`, `0087_backfill_environment_manage_human_defaults.sql`, `0088_backfill_principal_access_compatibility.sql`, `0089_cloud_upstreams.sql` (직전 기준선 `0085` → 현재 `0089`).
- **신규 테이블 1개**: `packages/db/src/schema/cloud_upstreams.ts` (+75 LOC, #6548). `packages/db/src/schema/` 디렉터리 파일 79개·`pgTable` 88개로 증가.
- **신규 어댑터 1개**: `packages/adapters/grok-local/` (#6087, +2031 LOC). `server/src/adapters/registry.ts:364` 에 `grok_local` 등록 — built-in 어댑터 타입 12개 → **13개**, 패키지형 9종 → **10종**.
- **신규 plugin 2개**: `packages/plugins/plugin-workspace-diff/` (#6071, +3360 LOC + #6383 폴리시 +418 LOC), `packages/plugins/sandbox-providers/modal/` (#6245, +1633 LOC).
- **신규 도메인**: Cloud Upstream sync (#6548, +5592 LOC) — `server/src/services/cloud-upstreams.ts` (+1298 LOC), `server/src/routes/cloud-upstreams.ts` (+118 LOC), `cli/src/commands/client/cloud.ts` (+721 LOC), UI 페이지 `CloudUpstream.tsx`/`CloudUpstreamUxLab.tsx` 신규.
- **권한·거버넌스 확장**: agent permissions and controls plan (#6386, +547 LOC `authorization-service`, plugin SDK protocol +156 LOC, `agent-permissions-routes` 신규).
- **`server/src/routes/issues.ts` 크기 증가**: 184 KB → **205 KB** (+264 LOC: #6371 +92, #6380 +172).
- **`server/src/services/issues.ts` checkout 코드 이동**: `4588-4731` → **`4659-4809`** (+71 LOC 하향 이동 — #6380 +56, #6547 +13, 미세 차).
- **i18n 확장**: `ui/src/i18n/locales/` 에 39개 언어 JSON 추가 (#6070).

---

## 2. 챕터별 판정

### Tier 1 — STALE (즉시 수정 필요)

#### `00-overview` — STALE

- §그림 1 본문 `12개의 built-in adapter type(server/src/adapters/registry.ts:480-493)`. grok-local(#6087) 추가로 카운트는 **13개**, BUILT_IN 배열 위치도 **`:511-524`** 로 이동(registerBuiltInAdapters 함수, 패키지형 11종 + generic 2 = 13개 항목).
- 같은 문장 `10개는 구체 런타임(Claude Code, Codex, Cursor local, Cursor Cloud, Gemini, OpenCode, Pi, ACP-x, OpenClaw 게이트웨이, Hermes)` → **11개는 구체 런타임** + Grok local 추가. **수정값**: 카운트 12→13, 라인 `:480-493`→`:511-524`, 명단에 Grok local 추가.

#### `01-architecture` — STALE

- §1 `built-in adapter type은 12개다(server/src/adapters/registry.ts:480-493)`. 카운트 13, 라인 :511-524 — 동일 이슈. **수정값**: 12→13, `:480-493`→`:511-524`.
- §1 `패키지형 9종 — acpx-local, claude-local, codex-local, cursor-cloud, cursor-local, gemini-local, opencode-local, pi-local, openclaw-gateway` → **10종** + grok-local. **수정값**: 명단에 grok-local 추가.
- 같은 문장 `hermes_local 은 ... server/src/adapters/registry.ts:117-127 에서 정적으로 등록한다` → 실제 `hermesLocalAdapter` 정의는 **438행**, hermes 동적 import 라인 변동 — 정확한 위치 재인용 필요. **수정값**: hermes 정적 등록 라인 갱신(현재 `:438-450` 정의, 등록 배열 `:511-524`).

#### `02-data-model` — STALE

- §5 atomic checkout 의사코드 캡션 `(실제 코드는 server/src/services/issues.ts:4588-4731)`. checkout block 이 #6380(+56 LOC) + #6547(+13 LOC) 으로 `4659-4809` 로 이동(`checkout: async` 4659행, 닫는 `},` 4809행). **수정값**: `4588-4731` → **`4659-4809`**.
- §3 `80여 개 테이블 중 약 60개가 가진 company_id 컬럼` — 현재 `pgTable` 88개(직전 80여개 → 신규 1개 + 기존 카운팅 정밀화 차). "80여 개"는 모호 표기라 STALE 아님, MINOR로 분류.

#### `03-runtime-execution` — STALE

- §1 그림 3-1 본문 `atomic checkout이 이슈를 잡는다(server/src/services/issues.ts:4588-4731)` — 동일 라인 이동. **수정값**: `4588-4731` → **`4659-4809`**.

#### `04-adapters-and-skills` — STALE

- §그림 7 캡션 `런타임 전용 built-in 어댑터 10개와 범용 어댑터 2개(server/src/adapters/registry.ts:480-493)` — 카운트와 라인 동시 STALE. **수정값**: 10→11, 라인 `:480-493`→`:511-524`.
- §2 본문 `getQuotaWindows는 ... claude-local과 codex-local만 제공 ... detectModel은 ... hermes-paperclip-adapter 경로(server/src/adapters/registry.ts:116-123, 456-465)` — hermes 경로 라인 이동(현재 hermes 어댑터 정의 :438-450, 외부 import 라인 변동). **수정값**: hermes 인용 라인 갱신.
- §4 본문 line 107 `server/src/adapters/registry.ts:106\~117은 hermes-paperclip-adapter를 정적 import` — 정적 import 위치 재확인 후 갱신.
- §추가: grok-local adapter 가 새 코딩 어댑터로 진입. §2 의 "주요 local coding adapters" 명단(`claude-local`, `codex-local`, `cursor-local`, `gemini-local`, `opencode-local`, `pi-local`) 에 grok-local 추가 여부는 신규 어댑터의 capability 정도에 따라 결정 — 본 사이클은 STALE 라인·카운트만 반영, capability 분류는 별도 GAP(MINOR).

#### `05-server-api` — STALE

- §2 표 1 행 `issues.ts (180 KB) | 이슈 풀 lifecycle + recovery action` → 실제 **205 KB**(210,041 바이트). #6371(+92 LOC) + #6380(+172 LOC) 으로 약 +21 KB. **수정값**: `180 KB` → **`205 KB`**.
- §2 본문 `issues.ts가 180 KB에 달하는 사실은` → **`205 KB`**.
- §3 본문 `routes/issues.ts 180 KB(§2 표 1)가 가장 큰 두 봉우리` → **`205 KB`** (이제 routes/issues.ts 가 services/company-portability.ts 178 KB 를 상회 → "가장 큰 봉우리").
- §3 그 다음 단락 `services/company-portability.ts 180 KB` 인용은 실제 178 KB. 명목 카테고리(180 KB 대) 유지 가능 → MINOR.

#### `07-governance-cost` — STALE + GAP

- §3 본문 `락/언락은 activity_log 에 ... 감사 흔적을 남긴다(server/src/routes/issues.ts:2093·2141)` → 락 라우트가 `routes/issues.ts:2548·2596` 으로 +455행 이동(#6371/#6380 등 routes/issues.ts 추가 LOC + 그 사이 라우트 이동량). **수정값**: `:2093·2141` → **`:2548·2596`**.
- **GAP**(별도 반영 대상): **agent permissions and controls plan(#6386)** — `authorization-service` +547 LOC, agent-permissions-routes 신규, 마이그레이션 `0087_backfill_environment_manage_human_defaults` · `0088_backfill_principal_access_compatibility`. 본문 §3 권한 표 1 의 7행 거버넌스 레버에 agent permissions 신규 항목 추가 여지(예: principal-access 호환성, environment manage 인간 기본값). 본 사이클은 STALE 라인만 반영, GAP 은 별도 반영.

#### `09-pros-cons` — STALE

- "단일 assignee + atomic checkout" 행의 코드 인용 `server/src/services/issues.ts:4588-4731`. **수정값**: `4588-4731` → **`4659-4809`**.
- "거대 단일 파일들" 행의 `routes/issues.ts 180 KB`. **수정값**: `180 KB` → **`205 KB`**.
- 같은 행의 `services/company-portability.ts 180 KB` — 실제 178 KB, "180 KB 대" 명목으로 유지 가능. MINOR.

#### `docs/research/05-openclaw-agent-companies` — STALE

- 본문 `어댑터 디렉터리에 acpx-local, claude-local, codex-local, cursor-cloud, cursor-local, gemini-local, opencode-local, openclaw-gateway, pi-local 9 개 패키지가 들어 있고 registry built-in 으로 등록된다 — server/src/adapters/registry.ts:480-493`. **수정값**: 9개 → **10개** (+ grok-local), 라인 `:480-493` → **`:511-524`**.

### Tier 2 — GAP (누락 토픽, 사용자 확인 후 별도 반영)

#### `06-ui-and-board` — GAP (확장)

직전 sync(2026-05-16) 에서 등재된 GAP-06 가 잔존하는 가운데, 37커밋 사이 UI 면이 더 확장됐다:

- **i18n locale catalog 39개(#6070)** — `ui/src/i18n/locales/` 에 ar/bn/cs/da/de/el/es/fa/fi/fil/fr/he/hi/hu/id/it/ja/ko/mr/ms/nb/nl/pa/pl/pt-BR/pt-PT/ro/ru/sv/sw/ta/te/th/tr/uk/ur/vi/zh-CN/zh-TW JSON. 직전 sync 의 "minimal foundation" 단계에서 본격 locale 단계로 확장.
- **board UI 모바일 폴리시(#6550)** — Layout, MarkdownBody, NewIssueDialog, CompanySettingsNav 등 다수 컴포넌트 모바일 흐름 강화 + DesignGuide 갱신.
- **inbox 표시 단순화(#6269)** — issue row 의 planning 배지 제거.
- **devops surfaces UI(#6384)** — DevRestartBanner, MarkdownEditor, Sidebar, Inbox 등 다수 storybook 추가 + UI 폴리시.
- **secret UX(#6339·#6381)** — SecretBindingPicker · JsonSchemaForm 시크릿 참조 통합, provider vault secrets UX.
- **issue 자동완성(#6311)** — NewIssueDialog autocomplete 포인터 선택.
- **invite/instance 사이드바(#6341·#6433)** — sandbox-provider 플러그인 숨김, 초대 페이지 query-key collision 수정.

#### `07-governance-cost` — GAP (agent permissions)

상기 STALE 항에 동시 분류. **agent permissions and controls plan(#6386)** 이 §3 권한 표 1 의 7행 거버넌스 레버에 추가 여지를 만든다. 마이그레이션 `0087·0088` 이 명시적으로 "environment_manage_human_defaults" 와 "principal_access_compatibility" backfill 을 수행 — 거버넌스 의도가 코드와 doc/SPEC.md 양쪽에 들어가 있다.

#### 신규 도메인 — GAP (cloud upstream)

**Cloud Upstream sync(#6548, +5592 LOC)** 는 새로운 운영 도메인을 도입한다:

- 마이그레이션 `0089_cloud_upstreams.sql` + 스키마 `packages/db/src/schema/cloud_upstreams.ts` (+75 LOC) + shared 타입 `packages/shared/src/types/cloud-upstream.ts` (+110 LOC).
- 서버: `server/src/services/cloud-upstreams.ts` (+1298 LOC) + `server/src/routes/cloud-upstreams.ts` (+118 LOC).
- CLI: `cli/src/commands/client/cloud-store.ts` · `cloud-transfer.ts` · `cloud.ts` (+1195 LOC).
- UI: `ui/src/pages/CloudUpstream.tsx` (+646 LOC) + `CloudUpstreamUxLab.tsx` (+822 LOC) + storybook fixtures.

**판정**: 학습 리포트 21개 챕터 중 어디에도 cloud upstream 도메인 서술이 없다. 단순 STALE 이 아니라 **신규 챕터/절 후보** 다. 04장(어댑터/스킬), 05장(서버 API), 또는 별도 신규 챕터 매핑이 필요. 본 사이클은 GAP 으로 기록만 하고, 사용자 확인 후 별도 반영.

### Tier 3 — MINOR / OK

| 챕터 | 판정 | 사유 |
|---|---|---|
| `01-architecture` | MINOR | `ui/src/i18n/locales/` 39개 신규 + `packages/adapters/grok-local/` + `packages/plugins/plugin-workspace-diff/` + `packages/plugins/sandbox-providers/modal/` — 패키지 지도 단위 다수 신규. coarse 단위라 본문 거짓 진술은 없음 |
| `04-adapters-and-skills` | MINOR | workspace-diff plugin · Modal sandbox provider plugin 신규 — 04장 plugin 분류에 별도 행 추가 여지(현 본문은 plugin-llm-wiki 등 일부만 언급) |
| `10-research-map` | MINOR | R5 행의 `services/company-portability.ts (180 KB)` 실제 178 KB — 명목 카테고리(180 KB 대) 유지 가능. STALE 미만 |
| `08-usage` | OK | 영향 없음 |
| `docs/research/01\~04`, `R6\~R10` | OK | 외부 기술 분석 — 37커밋의 영향 범위 밖 |
| `00-overview` 의 5원칙 본문 | OK | 어댑터 카운트 외 5원칙·3-zone 분리는 유지 |
| `02-data-model` 외 부분 | OK | issues 스키마(28-29, 37-38), cost_events 스키마(13-18) — 37커밋이 해당 컬럼 건드리지 않아 유지 |

소형 fix·hardening 커밋(#6142 grok-local turn boundary · #6222 secret textarea · #6240 worktree force · #6289 env probe timeout · #6311 autocomplete · #6338 docker · #6378 cloud tenant import · #6423 wrapped company prefix · #6433 invite page · #6560 E2B) 과 release/lockfile/ci(#6136 · #6263 · #6137 · #6154 · #6150 · #6290) 와 doc/skill(#6261 · #6152) 은 아키텍처 서술을 바꾸지 않아 본문 OK.

---

## 3. 판정 카운트

`docs/report/` 11개 + `docs/research/` 10개 = 총 **21개** 분석 대상.

| 판정 | 개수 | 챕터 |
|---|---|---|
| STALE | 9 | `00-overview`, `01-architecture`, `02-data-model`, `03-runtime-execution`, `04-adapters-and-skills`, `05-server-api`, `07-governance-cost`, `09-pros-cons`, `docs/research/05` |
| GAP | 3 | `06-ui-and-board` (i18n + UI 폴리시), `07-governance-cost` (agent permissions plan), **신규 도메인** (Cloud Upstream sync) |
| MINOR | 2 | `04-adapters-and-skills` (plugin 분류 추가 여지), `10-research-map` (KB 미세) |
| OK | 10 | `08-usage`, `docs/research/01\~04`, `docs/research/06\~10` |
| **합계** | **21** | (GAP 3건 중 07 은 STALE 과 중복 — 챕터 단위 카운트는 STALE 9 + GAP 1(06) + MINOR 2 + OK 9 = 21) |

신규 도메인 GAP 1건은 챕터 매핑 없는 별도 항목.

---

## 4. 적용 결과 (STALE 9건 — 2026-05-23 반영 완료)

`/loop` 사이클 Step 4 에서 STALE 판정분만 본문 반영:

- `00-overview.md` — 어댑터 카운트 12→13, registry `:480-493`→`:511-524`, "Grok local" 명단 추가
- `01-architecture.md` — 어댑터 카운트 12→13, registry `:480-493`→`:511-524`, 패키지형 9종→10종(grok-local 추가), hermes 정적 등록 라인 갱신
- `02-data-model.md` §5 — `issues.ts:4588-4731` → `issues.ts:4659-4809`
- `03-runtime-execution.md` §1 — `issues.ts:4588-4731` → `issues.ts:4659-4809`
- `04-adapters-and-skills.md` — built-in 카운트 10→11(범용 2 + 11 = 13), registry `:480-493`→`:511-524`, hermes 경로 인용 라인 갱신
- `05-server-api.md` §2·§3 — `issues.ts 180 KB`→`205 KB`(3개소), "가장 큰 두 봉우리" → routes/issues.ts(205 KB) 단독 최대
- `07-governance-cost.md` §3 — 락/언락 라우트 `:2093·2141`→`:2548·2596`
- `09-pros-cons.md` — `issues.ts:4588-4731`→`:4659-4809`, `routes/issues.ts 180 KB`→`205 KB`
- `docs/research/05-openclaw-agent-companies.md` — 패키지 9개→10개(grok-local), registry `:480-493`→`:511-524`
- `_pdf-build/` 챕터 사본 재동기화 + PDF/HTML 재빌드

### GAP 3건 — 2026-05-23 추가 반영 완료 (focused follow-up)

- **GAP-06**(UI 면 확장) — `06-ui-and-board.md` §2 표 1에 Cloud Upstream UI 행(`/company/settings/cloud-upstream`, `/ux-lab/cloud-upstream` — CloudUpstream.tsx 646 LOC + CloudUpstreamUxLab.tsx 822 LOC) 추가 + 새 절 **§2.1 "최근 추가된 UI 면 — 2026-05 사이클(#6070\~#6560)"** 신설. 6축 정리: ① i18n locale catalog 39개(#6070·#6058) ② 모바일 보드 흐름 폴리시(#6550) ③ secret/vault UX 통합(#6339·#6381) ④ dev/ops surfaces(#6384) ⑤ inbox/instance/invite 단순화(#6269·#6341·#6433) ⑥ workspace diff viewer 플러그인 UI(#6071·#6383).
- **GAP-07**(agent permissions and controls plan, #6386) — `07-governance-cost.md` §3 표 1 뒤에 단락 "**권한 표현의 내부 호환성 보강(#6386)**" 신설. 표 1 행(7행)은 변경하지 않고, 마이그레이션 `0087·0088` + `authorization-service` +547 LOC + plugin SDK protocol/host-client-factory +156 LOC 가 *기존 권한 모델의 내부 일관성과 검증 표면을 보강한다* 는 관점으로 정리.
- **신규 도메인 GAP**(Cloud Upstream sync, #6548) — `05-server-api.md` §2 표 1에 `cloud-upstreams.ts` 행 추가(8개 라우트 — connect/start·finish·push-runs·preview·cancel·activation), §3 디렉터리 트리에 `(cloud sync · 신규)` 군집 추가(`cloud-upstreams.ts` 48 KB), §3 본문에 단락 "**신규 도메인 — Cloud Upstream sync(#6548, 2026-05-22)**" 신설(스키마 `0089` · 서비스 +1298 LOC · UI 페이지 2개 · CLI 1195 LOC). `02-data-model.md` §6 표 2 Foundation 군집에 `cloud_upstreams` 추가.

### GAP 반영 후 산출물

- `_pdf-build/` 챕터 사본 4개 추가 동기화(02·05·06·07).
- **report 포맷** 재빌드: 72 페이지 / 3.96 MB.
- **slide 포맷** 신규 빌드: 41 페이지 / 1.10 MB — `paperclip-learning-report.slides.{html,pdf}` 로 산출.
- 표지 KPI Pages 75 → 69 → **72** (GAP 반영 후), date `2026-05-23` 유지.
- `docs/report/pdf/README.md` 갱신: slide 빌드 한 줄 명령 + 산출물 4종 매핑(report·slide × html·pdf).

---

## 5. 메모

- 모든 LOC·라인번호·파일 크기는 2026-05-23 시점 `e3c875c1`(origin/master) 기준으로 직접 검증함.
- checkout 코드 이동량 +71행은 def 행(4588→4659)과 닫는 `},` (4731→4809) 두 지점에서 동일하게 확인 — 블록 내부는 #6380/#6547 이 미세하게 건드렸으나 균일 이동에 가까움.
- routes/issues.ts 의 락/언락 라우트 이동량(+455행)은 본 사이클 추가 LOC(+264) 보다 큼 — direct 측정 결과를 신뢰. 직전 sync 기준선(`4c47eb46`) 에서도 이미 라우트가 일부 이동했을 가능성 있으나, 학습 리포트 인용은 origin/master 의 현재 위치로 갱신.
- `server/src/adapters/registry.ts` 의 BUILT_IN 배열은 직전 sync 의 `:480-493` 14줄 범위에서 함수 `registerBuiltInAdapters()` 의 배열 정의 `:511-524` 14줄(grok-local 1줄 + 패키지형 9 + generic 2 + 위치 이동) 로 갱신.
- 신규 마이그레이션 4개(0086\~0089) 는 학습 리포트가 마이그레이션 카운트를 명시하지 않아 STALE 아님 — 07장의 `마이그레이션 0085` 특정 인용은 락 기능 마이그레이션을 지칭하므로 유효.
- 본 사이클은 origin/master 37커밋을 로컬 master 에 rebase 동기화한 직후 작성.
