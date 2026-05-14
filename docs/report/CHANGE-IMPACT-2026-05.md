# Change-Impact Report — 2026-05-13

**대상 범위**: master 브랜치 31개 커밋 (`0e1a5828` → `b947a7d7`)  
**비교 대상**: 기존 학습 리포트 (`docs/report/00\~10`, `docs/research/01\~10`)  
**작성일**: 2026-05-13  
**판정 기준**: STALE(즉시 수정 필요) / GAP(누락 토픽) / MINOR(소폭 추가) / OK(유지)

---

## 1. 들어오는 변경의 전체 요약

| # | PR | 제목 | 영향 영역 |
|---|---|---|---|
| 1 | #5429 | Secrets provider vaults and remote import | server/api, ui, cli, db |
| 2 | #5599 | Source-scoped recovery actions | server/services, db, ui |
| 3 | #5664 | cursor_cloud adapter (Cursor SDK + Cloud Agents API v1) | packages/adapters |
| 4 | #5688 | exe.dev sandbox provider plugin | packages/plugins |
| 5 | #5687 | Cloudflare sandbox provider plugin | packages/plugins |
| 6 | #5580 | Daytona sandbox provider plugin | packages/plugins |
| 7 | #5716 | LLM Wiki plugin host support + package | packages/plugins, server |
| 8 | #5821 | Local plugin development workflow 개선 | cli, server, docs |
| 9 | #5585 | Shared sidebar section controls | ui/components |
| 10 | #5703/#5764 | Routine properties panel / changelog | ui, release |
| 11 | misc | Heartbeat 견고화, runtime probes, lockfile refresh | server/services |

**규모**: 396 파일 / +58,532 / -1,522 (line)

**검증된 핵심 신규 코드**:

- `server/src/services/issue-recovery-actions.ts` — **295 LOC** (신규)
- `server/src/services/plugin-managed-skills.ts` — **359 LOC** (신규)
- `packages/db/src/schema/issue_recovery_actions.ts` — 신규 테이블 스키마
- `packages/db/src/schema/company_secret_provider_configs.ts` — 신규
- `packages/db/src/schema/company_secret_bindings.ts` — 신규
- `packages/db/src/schema/secret_access_events.ts` — 신규
- `packages/db/src/schema/company_secret_versions.ts` — 신규 (시크릿 버전 관리)
- `packages/db/src/migrations/0082_dry_vision.sql` — `company_secret_bindings` + `secret_access_events`
- `packages/db/src/migrations/0083_company_secret_provider_configs.sql`
- `packages/db/src/migrations/0084_issue_recovery_actions.sql`
- `ui/src/pages/Secrets.tsx` — **2,155 LOC** (대폭 확장)
- `ui/src/pages/secrets/ImportFromVaultDialog.tsx` — **1,477 LOC** (신규)
- `cli/src/commands/client/secrets.ts` — **501 LOC** (신규/확장)
- `packages/adapters/cursor-cloud/` — 신규 어댑터 패키지
- `packages/plugins/sandbox-providers/{cloudflare,daytona,exe-dev}/` — 신규 3종 (`e2b`는 기존 — 기존 1종 + 신규 3종 = 총 4종)
- `packages/plugins/plugin-llm-wiki/` — 신규 플러그인 패키지 (wiki, agents, skills, templates 포함)

**registry 등록 현황 (`server/src/adapters/registry.ts:480-493`)**:

런타임 전용 10종(`acpx`, `claude`, `codex`, `opencode`, `pi`, `cursor-cloud`, `cursor-local`, `gemini`, `openclaw-gateway`, `hermes-local`) + 범용 2종(`process`, `http`) = **총 12종 built-in**. 직전(pre-pull) 기준 11종 → cursor-cloud 추가로 12종.

---

## 2. 챕터별 영향 판정

### 2.1 `docs/report/` (코어 리포트 11개)

| 챕터 | 판정 | 주요 영향 |
|---|---|---|
| 00-overview.md | **OK** | 아키텍처 차원의 변화 없음. V1 범위 안 항목 이행 중. |
| 01-architecture.md | **MINOR** | built-in adapter 11 → 12 (cursor-cloud 추가). sandbox provider 플러그인 3종 외부화 사례 추가. |
| 02-data-model.md | **MINOR** | 신규 테이블 4개: `issue_recovery_actions` (0084), `company_secret_provider_configs` (0083), `company_secret_bindings`·`secret_access_events` (0082). 기존 `company_secrets`·`company_secret_versions` 컬럼 확장. 12개 핵심 ER 자체는 불변. |
| 03-runtime-execution.md | **MINOR** | §6 보수적 회복이 issue별 source-scoped로 세분화. Heartbeat 시퀀스(그림 5) 자체는 불변. |
| 04-adapters-and-skills.md | **STALE** | 본문이 "런타임 9 + generic 2 = 11"인데 현재 registry는 "런타임 10 + generic 2 = 12" (cursor-cloud 추가). 그림 7에도 cursor-cloud 미반영. 플러그인 sandbox provider 범주 자체가 누락 (기존 `e2b` + 신규 `cloudflare`/`daytona`/`exe-dev`). `plugin-managed-skills` 서비스 미언급. |
| 05-server-api.md | **MINOR** | secrets 라우트 그룹 확장, recovery-actions 라우트 신규. 도메인 서비스 트리에 `issue-recovery-actions.ts`, `plugin-managed-skills.ts` 누락. |
| 06-ui-and-board.md | **MINOR** | `SidebarSection` 추출, `SidebarAgents`/`SidebarProjects` 분할 (#5585). `IssueRecoveryActionCard`(537 LOC) 신규 컴포넌트. `Secrets.tsx`가 2,155 LOC로 비대. |
| 07-governance-cost.md | **OK** | 예산/승인/활동 로그 흐름 동일. recovery action이 implicit activity source가 됨. |
| 08-usage.md | **MINOR** | CLI `paperclipai secrets` 서브커맨드 신설(501 LOC). 로컬 플러그인 개발 workflow 문서 갱신. |
| 09-pros-cons.md | **MINOR** | 강점 #4(보수적 회복) 근거 파일 한 줄 보강. 구조 변화 없음. |
| 10-research-map.md | **OK** | 매핑 구조 불변. R1(orchestration)/R4(headless agents) 사례 추가만. |

### 2.2 `docs/research/` (리서치 10개)

| 파일 | 판정 | 비고 |
|---|---|---|
| 01-agent-orchestration-control-plane.md | OK | 외부 표준 정리, 코드 변경과 무관 |
| 02-model-context-protocol.md | OK | 동일 |
| 03-claude-code-cli.md | OK | plugin dev workflow 사례 보강 가능(선택) |
| 04-headless-coding-agents.md | **MINOR** | cursor_cloud(Cursor SDK + Cloud Agents API v1) 사례 추가 가능 — 한 단락 |
| 05-openclaw-agent-companies.md | OK | |
| 06-drizzle-pglite.md | OK | |
| 07-pnpm-monorepo.md | OK | plugin 패키지가 추가됐으나 워크스페이스 구조는 동일 |
| 08-react-vite-shadcn.md | OK | |
| 09-sse-heartbeat-patterns.md | OK | heartbeat 견고화는 구현 디테일, 외부 패턴 변화 없음 |
| 10-llm-cost-budgeting.md | OK | |

---

## 3. 우선순위 권고

### Tier 1 — 정합성 깨짐 (즉시 수정 권장)

**04-adapters-and-skills.md (STALE)**

- 그림 7에 `cursor-cloud` 행 추가 — getQuotaWindows / sessionCodec / live event 컬럼 채우기
- 표 1 카운트 갱신: "런타임 9 + generic 2 = 11" → "런타임 10 + generic 2 = 12" (검증: `server/src/adapters/registry.ts:480-493`)
- 플러그인 sandbox provider 섹션 신설 — *현존 범주*는 4종 (`e2b` 기존 + `cloudflare`·`daytona`·`exe-dev` 신규 3종). 신규/기존 구분해 표기
- `server/src/services/plugin-managed-skills.ts:1-359` 언급 — 플러그인이 스킬을 등록·관리하는 새 메커니즘

### Tier 2 — 사실 보강 (한 단락 추가로 끝남)

**02-data-model.md (MINOR)**

- 표 2 "Plugins / extension" 행에 `issue_recovery_actions` 추가
- Secrets/Vault 섹션 보강 (#5429): `company_secret_provider_configs`, `company_secret_bindings`, `secret_access_events`, `company_secret_versions` 4종 + 기존 `company_secrets` 컬럼 확장. 마이그레이션 `0082_dry_vision.sql`, `0083_company_secret_provider_configs.sql`, `0084_issue_recovery_actions.sql` 함께 인용

**03-runtime-execution.md (MINOR)**

- §6 회복 규칙에 "source-scoped recovery action" 한 단락 추가 — `kind` × `status` × `fingerprint` 모델
- 근거: `server/src/services/issue-recovery-actions.ts:1-295`

**05-server-api.md (MINOR)**

- 코드 1(`services/` 트리)에 `issue-recovery-actions.ts`, `plugin-managed-skills.ts` 추가
- 라우트 표에 `secrets/vaults`, `issues/:id/recovery-actions` 행 추가

**06-ui-and-board.md (MINOR)**

- 사이드바 컴포넌트 분할(#5585) 1~2줄 기록
- `IssueRecoveryActionCard.tsx:1-537` 컴포넌트 언급
- Secrets 페이지 비대화(2,155 LOC) → 향후 약점 후보로만 기록

**01-architecture.md (MINOR)**

- 그림 2의 어댑터 카운트 갱신
- 플러그인 sandbox provider 외부화 사례 한 줄 추가

**08-usage.md (MINOR)**

- 코드 2(CLI 명령)에 `pnpm paperclipai secrets <sub>` 블록 추가

**09-pros-cons.md (MINOR)**

- 강점 #4 근거에 `issue-recovery-actions.ts` 추가

### Tier 3 — 손대지 않아도 됨

`00-overview`, `07-governance-cost`, `10-research-map`, `docs/research/01\~03`, `docs/research/05\~10` (9개). `docs/research/04-headless-coding-agents.md` 만 Tier 2 (cursor-cloud 한 단락).

---

### 판정 카운트 (정합성 표)

`docs/report/` 11개 + `docs/research/` 10개 = 총 **21개** 분석 대상.

| 판정 | 개수 | 챕터 |
|---|---|---|
| STALE | 1 | `04-adapters-and-skills` |
| MINOR | 8 | `01-architecture`, `02-data-model`, `03-runtime-execution`, `05-server-api`, `06-ui-and-board`, `08-usage`, `09-pros-cons`, `research/04-headless-coding-agents` |
| OK | 12 | `00-overview`, `07-governance-cost`, `10-research-map`, `research/01\~03`, `research/05\~10` |
| **합계** | **21** | |

## 4. 작업 견적

| 챕터 | 예상 작업 | 비고 |
|---|---|---|
| 04 (STALE) | 30\~45분 | 그림 7 + 본문 + sandbox provider 신규 섹션, 검증 포함 |
| 02 / 03 / 05 / 06 (MINOR ×4) | 각 10\~15분 | 한두 단락 추가 |
| 01 / 08 / 09 (MINOR ×3) | 각 5\~10분 | 한 줄~한 단락 추가 |
| research/04 (MINOR ×1) | 10\~15분 | cursor-cloud 사례 단락 |
| **총 (코어 7 + 04장 1 + research 1 = 9챕터)** | **~3시간** | 검증·정합성 점검 포함 |

---

## 5. 메모

- 모든 LOC와 경로는 2026-05-13 시점 `b947a7d7` 기준으로 직접 검증함.
- 본 리포트는 *triage* 문서이며, 본문 수정 전 사용자 확인을 전제로 작성.
- `_pdf-build/`, `pdf/`, `assets/` 산출물은 본문 수정 후 재빌드 대상.
