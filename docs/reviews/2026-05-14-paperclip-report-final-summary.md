# Paperclip 학습 리포트 합의 루프 — 최종 보고

**완료일**: 2026-05-14
**대상**: `docs/report/00-overview.md` \~ `docs/report/10-research-map.md` (11개 챕터)
**리뷰어**: Codex CLI peer reviewer (gpt-5.5 high, `paperclip:codex` tmux 윈도우)
**모드**: 자율 합의 루프 — `===CODEX_FINAL_APPROVAL===` 종료 신호 기반

---

## 1. 결과 요약

11개 챕터 **전수 합의 완료** (`===CODEX_FINAL_APPROVAL===`). REJECT 항목 0건. 누적 ACCEPT 64건.

| 챕터 | 파일 (LOC) | 합의 라운드 | r1 ACCEPT | 누적 ACCEPT | 주요 정정 |
|---|---|---|---|---|---|
| Ch00 | `00-overview.md` (87) | r3 | 10+ | 11+ | adapter 11→12 / 어댑터 계약 2층 / README 6 문제·7 페르소나 |
| Ch01 | `01-architecture.md` (117) | r3 | 8 | 12+ | 표 1 의존 재작성 / dev 포트 detect-port / `pnpm build` vs `-r build` |
| Ch02 | `02-data-model.md` (142) | r2 | 6 | 6 | pgTable 86 / `cost_events.agent_id` notNull / atomic checkout `:3593-3736` |
| Ch03 | `03-runtime-execution.md` (127) | r4 | 5 | 9 | 어댑터 계약 / source-scoped recovery action / 7-pass reconciliation |
| Ch04 | `04-adapters-and-skills.md` (157) | r5 | 4 | 11 | cursor-cloud / sandbox provider 4종 / plugin-managed-skills |
| Ch05 | `05-server-api.md` (180) | r4 | 5 | 10 | WS `authorizeUpgrade` / LIVE_EVENT_TYPES 9 / recovery-actions / agent local JWT |
| Ch06 | `06-ui-and-board.md` (147) | r5 | 6 | 13 | React 19 / Tailwind 4 / router 7 / 라우팅 표 재작성 / `IssueRecoveryActionCard` |
| Ch07 | `07-governance-cost.md` (145) | r4 | 5 | 8 | approvals 12 컬럼 / hard stop 3 scope / activity_log 4 actor / raise vs keep_paused |
| Ch08 | `08-usage.md` (201) | **r3** | **6** | **10** | 첫 화면 입력 2 항목 / `/agents/new` 표 정정 / Docker compose 풀스택 / authenticated env 보강 / board-claim 조건부 |
| Ch09 | `09-pros-cons.md` (99) | **r3** | **6** | **7** | 어댑터 계약 두 층 / atomic checkout `:3593-3736` / 보수적 회복 인용 보강 / hard stop 3 메커니즘 / 고급 거버넌스 |
| Ch10 | `10-research-map.md` (148) | **r2** | **4** | **4** | recovery 4-pass → 7-pass / workspace 8개 → 24개 / 어댑터 계약 3층 / 본문 그림 참조 평문화 |

총 `docs/report/` 본문 LOC = **1,550** (11개 합), `docs/reviews/` 산출물 **64개 파일**.

## 2. 합의된 코드 진실 (HANDOFF.md §3 누적)

이 절은 11개 챕터의 모든 라운드에서 *코드 검증으로 확정된* 사실 모음. 다음 학습 리포트가 재참조할 수 있는 단일 진실 묶음.

**어댑터**
- 내장 어댑터 12개 = 런타임 10 + generic 2 (`server/src/adapters/registry.ts:480-493`).
- 어댑터 계약 두 층: 장기 SPEC `invoke/status/cancel` (`doc/SPEC.md:207-215`) vs 현재 구현 `execute`+`testEnvironment` 필수(`packages/adapter-utils/src/types.ts:349-431`).
- adapter package 진입점 4개: `.` / `./server` / `./ui` / `./cli` (`packages/adapters/claude-local/package.json:15-20`).
- sandbox provider plugin 4종 (e2b / cloudflare / daytona / exe-dev).
- plugin-managed-skills capability (`server/src/services/plugin-managed-skills.ts:1-359`).

**Data model**
- pgTable 86개 / 스키마 파일 78개 / `companyId` 보유 파일 63개.
- 마이그레이션: `packages/db/src/migrations/` (`drizzle.config.ts:5`).
- issue status 7개: backlog/todo/in_progress/in_review/done/blocked/cancelled. terminal: done/cancelled.
- `cost_events`: `agent_id` notNull, 나머지 4 FK nullable, `cached_input_tokens` / `billing_code` / `project_id` / `heartbeat_run_id` 컬럼.
- atomic checkout `server/src/services/issues.ts:3593-3736`; 조건부 UPDATE `:3634-3654`, adopt/conflict `:3682-3736`.
- approvals 12 컬럼; `issue_approvals` 보조 링크.
- `activity_log.actor_type` 4분류: system / user / agent / plugin.
- budget hard stop scope 3종: company / agent / project, `cancelWorkForScope` 훅 (`server/src/services/heartbeat.ts:9431-9456`).
- `budget_incidents` 테이블 (`packages/db/src/schema/budget_incidents.ts:7-42`).
- `companies.issue_prefix` + `issue_counter` 회사별 단조 증가, prefix는 서버가 회사명에서 자동 파생 (`server/src/services/companies.ts:140-149`).

**Runtime / 회복**
- Startup reconciliation 7-pass: reap → promote scheduled retries → resume queued → stranded → liveness → silent → productivity reviews (`server/src/index.ts:676-780`).
- source-scoped recovery action: `issue_recovery_actions` (`packages/db/src/schema/issue_recovery_actions.ts:61-66`).
- 보수적 회복 4문구: *preserve ownership / retry once / explicit recovery action / escalate visibly* (`doc/execution-semantics.md:407-422`, `doc/SPEC-implementation.md:434-442`).
- silent active-run watchdog → recovery action 표면화; issue-backed 평가는 `stale_active_run_evaluation`.
- hard stop 세 메커니즘: `getInvocationBlock` / `withAgentStartLock` / `cancelBudgetScopeWork`.

**Server / Realtime / Auth**
- WS live events: HTTP upgrade `authorizeUpgrade` 별도 인증 (local_trusted / Better Auth / agent API key bearer). board API key는 WS bearer 미지원.
- `LIVE_EVENT_TYPES` 9개 (`packages/shared/src/constants.ts:596-606`).
- REST credential 5종: board user (Better Auth) / local-board / board API key / agent API key / **agent local JWT** (단명, `server/src/agent-auth-jwt.ts`).
- Plugin SDK hook 6 카테고리: lifecycle / data namespaces / events·webhooks / UI extensions / adapter plugins / managed resources.

**UI**
- Vite 6 + React 19 + Tailwind 4 + shadcn/ui + Radix UI + TanStack Query 5 + react-router-dom 7.
- 어댑터 매니저: `/instance/settings/adapters`; 인스턴스 플러그인 `/instance/settings/plugins(/:pluginId)`; 회사 스코프 플러그인 `/plugins/:pluginId` · `/:pluginRoutePath/*`.
- `Secrets.tsx` 2,155 LOC; `IssueRecoveryActionCard.tsx` 537 LOC.
- `Sidebar.tsx` 분할 (`SidebarSection` / `SidebarAgents` / `SidebarProjects`).
- `/dashboard/live` = `ActiveAgentsPanel` (활성/최근 에이전트 run; `/activity` 와 별도).
- 첫 화면 입력 2 항목: `Company name` + `Mission/goal` (`ui/src/components/OnboardingWizard.tsx:681,700`).
- `/agents/new` 입력 7 항목: name · title · role · reports_to · adapter_type · adapter_config · runtime_config · (옵션) desiredSkills. `icon` / `capabilities` / `budgetMonthlyCents` 는 생성 후 edit 화면.

**호스팅 / Docker**
- `docker/docker-compose.yml` = db (postgres:17-alpine) + server (Paperclip Dockerfile) 풀스택.
- 단독 `docker run` 도 `BETTER_AUTH_SECRET` 필수 — Dockerfile 기본 `PAPERCLIP_DEPLOYMENT_MODE=authenticated` (`Dockerfile:76`).
- `authenticated/public` 모드 필수 env: `PAPERCLIP_DEPLOYMENT_MODE` + `PAPERCLIP_DEPLOYMENT_EXPOSURE` + `BETTER_AUTH_SECRET` + `PAPERCLIP_PUBLIC_URL` (`cli/src/checks/deployment-auth-check.ts:28-60`, `server/src/config.ts:160-180`).
- board-claim URL: authenticated 모드 + instance admin이 `local-board` 한 명일 때만 부팅 경고 (`doc/DEPLOYMENT-MODES.md:115-126`, `server/src/board-claim.ts:43-66`).

**Workspace / 거버넌스**
- pnpm workspace 24 패키지: `packages/*`, `packages/adapters/*` (9), `packages/plugins/*`, `packages/plugins/examples/*`, `server`, `ui`, `cli`.
- 회사 멤버십(`company_memberships`) + 휴먼 role 4단계 owner/admin/operator/viewer.
- `approvalsNeeded > 1` 거부, multi-member boards / advanced governance = Not V1 (`doc/SPEC.md:489-493`).

**Overview / V1 scope**
- README 문제 6개 / 페르소나 7개 (`README.md:131-140`, `:70-78`).
- PRODUCT.md 5원칙 ≠ SPEC §13 9원칙. 리포트 표 1의 5개는 두 문서를 합쳐 학습용으로 추린 것.
- V1 Out: KB · 매출 회계 = future plugin; ClipHub · multi-board governance · self-healing = V1 밖.
- issue `goal_id`/`parent_id` nullable + `issue-goal-fallback.ts`.

## 3. DEFER 보존 항목

라운드 전체에서 DEFER 로 합의된 항목 — 사실 오류는 아니나 향후 정합화 대상.

- **라인 범위 ±1\~2줄 nit** — 각 챕터 라운드마다 반복적으로 등장. substantive 한 사실 오류 위주로 ACCEPT 정책 운영.
- **영문 용어 취향** — `org tree`, `Augmented LLM`, `Orchestrator-workers` 등 외래어 표기. 본문 가독성을 위해 그대로 유지.
- **코드/표 캡션 문체** — `**코드 N. ...**`, `**그림 N. ...**` 의 부제 형식 차이 (예: 출처 표기 위치, 부제 길이).
- **외부 인용 그림 글로벌 번호 정합화** — Ch00 1-A → 그림 2, Ch02 4-A/4-B → 그림 5/6 등 통합 완료. 그러나 챕터 간 글로벌 번호 충돌(예: 같은 "그림 2" 가 Ch01·Ch02 양쪽에 존재) 정합화는 향후 일괄 작업.
- **거대 단일 파일 분해** — `services/company-portability.ts` 180 KB, `routes/issues.ts` 168 KB 등 — Paperclip 코드 레벨 리팩토링 영역으로 리포트 범위 밖.

## 4. OUT-OF-SCOPE (이번 합의에 포함되지 않은 영역)

- **그림 SVG / PNG 자체의 시각 디자인 검증** — 본문 캡션·인용 형식만 검토.
- **PDF 변환 시점의 페이지 분할·페이지 번호** — `format: report / target: A4` front-matter만 검증.
- **`docs/research/` 디렉터리 (R1\~R10 본문 자체)** — Ch10 표 1 의 매핑 항목만 검증, 리서치 본문은 별도 합의 대상.
- **Paperclip 코드 자체 수정** — 변경 0건. 산출물은 `docs/report/` 와 `docs/reviews/` 에 한정.

## 5. 운영 메모

- **라운드 캡 없음** 운영이 효과적이었다. 6 챕터(00·03·04·05·06·07) 는 4\~5 라운드, 4 챕터(01·02·08·09)는 2\~3 라운드, Ch10 은 2 라운드로 자연스럽게 수렴.
- **자율 모드** (중간 보고 없이 final-summary 까지 일괄 진행) 는 사용자가 폴링 narration 만 받고 챕터별 결과 검토를 미루는 선호와 맞았다.
- **handout-then-clear** 가 한 차례도 발동되지 않았다. Codex 자체 컨텍스트 압축이 라운드 사이에 자동으로 일어나 Context 가 67\% \~ 78\% 대에서 회수됨.
- **자연어 승인 거부 규칙** (오직 `===CODEX_FINAL_APPROVAL===` 만 종료 신호) 이 *Codex 가 자연스럽게 "최종 승인" 이라고 말하면서도 잔존 위반을 가진* 케이스를 막아 줬다. Ch09 r2 가 그 예 — "단일 보드" 잔존 1곳을 명시적으로 표면화.
- **paste-buffer + Enter 두 번** 패턴 — Ch10 라운드들 중 일부에서 Enter 한 번 만으로 submit 안 된 케이스 있었음. 첫 Enter 후 capture-pane 으로 `Working` 표시 확인 → 안 보이면 Enter 한 번 추가 가 안정적인 흐름.

## 6. 다음 학습 리포트가 이어받을 출발점

- `docs/reviews/HANDOFF.md` §3 합의된 사실 묶음은 *2026-05-14 시점* 합의. 이후 코드 변경 시 stale 검사를 라운드 1에서 첫 단계로 수행.
- `docs/report/00-overview.md` \~ `10-research-map.md` 가 다음 리포트의 *진입 지도* 로 사용 가능.
- DEFER 보존 항목(특히 외부 인용 그림 글로벌 번호) 은 다음 사이클에서 일괄 정합화 가능.

---

**합의 루프 끝.**
