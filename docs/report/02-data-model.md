---
format: report
target: A4
topic: Paperclip 데이터 모델 (02)
---

# Data Model — 80개 이상 테이블 한눈에 보기

## 1. 데이터 모델 한눈 — 핵심 ER

`packages/db/src/schema/` 에는 **88개의 `pgTable()` 정의가 79개 `pgTable` 보유 스키마 파일** 에 분산되어 있다(`packages/db/src/schema/index.ts:1-79`). 모두를 한 그림에 담는 것은 무의미하므로, 보드 운영 흐름을 따라가는 데 필수적인 12개 이상 테이블을 추려 그림 4로 묶었다.

**그림 4. 핵심 ER 다이어그램 — issues를 중심으로 한 12개 이상 테이블**

![](assets/fig-04-er-core.svg)

가운데 빨간 박스가 **`issues`** — 단일 작업 단위(unit of work)다. 위쪽 회색 박스 4개(`agents` · `goals` · `projects` · `approvals`)는 모두 `companies`에서 내려온다. 아래쪽 청색 박스 그룹(`cost_events` · `budget_policies` · `budget_incidents` · `activity_log`)은 비용·거버넌스 레이어다. 이 그림은 **핵심 회사 도메인 데이터가 `company_id`로 스코프** 되며, 이슈 한 개가 어떻게 cost·activity·heartbeat와 연결되는지를 보여 준다. 핵심 관찰점은 세 가지다. 첫째, `issues` 박스 안의 `assignee_agent_id XOR assignee_user_id` 한 줄은 한 이슈를 *오직 한 명의* 에이전트 또는 사람이 책임지는 단일 assignee 규칙(§4)의 시각적 증거다. 둘째, `issues`는 `checkout_run_id` 와 `execution_run_id` 두 컬럼으로 `heartbeat_runs.id` 를 *직접* 참조하고(`packages/db/src/schema/issues.ts:37-38`), `activity_log.run_id` 는 실행 중 발생한 감사 이벤트를 같은 run 축으로 묶는 보조 다리다. 셋째, `cost_events` 는 `agent_id` 가 필수이고(`packages/db/src/schema/cost_events.ts:13-18`), `issue_id` · `project_id` · `goal_id` · `heartbeat_run_id` 네 개 외래키만 nullable이라 동일 비용 사건을 *다중 축으로 선택적* 으로 귀속시킨다.

## 2. 단일 불변항: company-scoped

`AGENTS.md` Core Engineering Rule #1이 강제하는 단일 불변항은 명료하다 — *"Every domain entity should be scoped to a company and company boundaries must be enforced in routes/services."* 88개 `pgTable()` 정의 중 67개가 `company_id` 컬럼을 들고 있다. 다만 일부 플러그인/인스턴스/파생 테이블은 `plugin_id`, `user_id`, 상위 company-scoped FK 등 다른 축으로 귀속된다. 라우트 레이어의 미들웨어가 모든 mutating call에 대해 회사 멤버십·권한을 검사한다.

## 3. 7개 핵심 테이블 — 표로 본 컬럼 요약

**표 1. 기둥 테이블 7개의 핵심 컬럼**

| 테이블 | 핵심 컬럼 | 무엇을 의미 |
|---|---|---|
| `companies` | `id`, `status`, `issue_prefix`, `issue_counter`, `budget_monthly_cents`, `spent_monthly_cents`, `require_board_approval_for_new_agents`, `brand_color` | 회사 = 모든 데이터의 루트. issue 번호도 회사별 카운터. |
| `agents` | `company_id`, `reports_to (FK→agents)`, `adapter_type`, `adapter_config jsonb`, `runtime_config jsonb`, `default_environment_id`, `budget_monthly_cents`, `last_heartbeat_at` | 트리 형태 org. adapter 타입과 jsonb 설정이 에이전트의 본체. |
| `issues` | `company_id`, `parent_id (self FK)`, `project_id`, `issue_number` + `identifier` (예: `PAP-123`), `status`, `assignee_agent_id` ⊕ `assignee_user_id`, `checkout_run_id`, `execution_run_id`, `execution_policy jsonb`, `billing_code`, `origin_kind/origin_id/origin_run_id` | 단일 이슈 모델. assignee는 한 명뿐이다. blocker는 별도 relation/`PATCH /issues/:id`의 `blockedByIssueIds` payload로 다룬다. |
| `cost_events` | `company_id`, `agent_id`, `issue_id?`, `project_id?`, `goal_id?`, `heartbeat_run_id?`, `provider`, `biller`, `billing_type`, `model`, `input_tokens`, `cached_input_tokens`, `output_tokens`, `cost_cents`, `billing_code`, `occurred_at` | 토큰·달러 모두 기록. `agent_id` 는 필수, issue/project/goal/heartbeat run 축은 선택적으로 붙어 다중 귀속을 만든다. |
| `heartbeat_runs` | `company_id`, `agent_id`, `invocation_source`, `trigger_detail`, `status`, `started_at/finished_at`, `wakeup_request_id?`, `exit_code`, `usage_json`, `result_json`, `session_id_before/after`, `log_store/log_ref/log_bytes`, `stdout_excerpt/stderr_excerpt` | 한 회차 실행. `issues.checkout_run_id` / `execution_run_id` 가 이 테이블을 직접 참조하고, `activity_log.run_id` 도 같은 run 축에 묶인다. 비용·로그·복구의 기준 단위. |
| `approvals` | `company_id`, `type`, `requested_by_agent_id?`, `requested_by_user_id?`, `payload jsonb`, `status`, `decision_note`, `decided_by_user_id?`, `decided_at?` | 보드 승인 게이트(고용·전략·budget hard 등). 대상은 `type` 과 `payload` 로 표현된다. |
| `activity_log` | `company_id`, `actor_type` (`system`/`user`/`agent`), `actor_id`, `action`, `entity_type`, `entity_id`, `details jsonb`, `agent_id?`, `run_id?` | 모든 mutating action의 불변 감사 로그. `run_id`가 heartbeat_runs ↔ issues를 잇는 다리. |

표 1 을 한 묶음으로 보면 *7개 테이블이 곧 control plane 의 7개 어휘* 라는 점이 드러난다 — 회사(`companies`)가 모든 데이터의 루트, 그 안에 *행위자* (`agents`)와 *작업 단위* (`issues`)가 살고, 모든 *행위* 는 `cost_events` 와 `activity_log` 가 두 채널로 기록하며, *판단* 은 `approvals` 에 모이고, *실행 회차* 는 `heartbeat_runs` 가 끊어 준다. `cost_events` 의 컬럼 구성에서 중요한 점은 `cached_input_tokens` 가 별도 컬럼으로 존재한다는 것이다. 이는 Anthropic·OpenAI·Gemini 가 모두 채택한 **prompt caching** 가격 모델을 회계까지 끌어 올린 결과다(자세한 내용은 [docs/research/10-llm-cost-budgeting.md](../research/10-llm-cost-budgeting.md)).

## 4. issue assignee의 XOR 규칙

`issues`의 `assignee_agent_id`와 `assignee_user_id`에는 **동시에 값이 들어갈 수 없다** — `doc/execution-semantics.md` §2가 강한 불변조건으로 명시한다. 이유는 깔끔하다. 에이전트가 가진 이슈는 heartbeat scheduler의 실행 루프에 들어가지만, 사람이 가진 이슈는 들어가지 않는다. 한 자리에 둘이 동시에 있으면 회복 규칙을 단순하게 적을 수 없다.

이 분리 덕분에 **`status='in_progress'`가 에이전트 소유면 "라이브 실행이 보장돼야 한다"** 는 강한 계약을 둘 수 있다. 사람 소유 이슈는 같은 status라도 heartbeat가 깨지지 않는다.

## 5. atomic checkout — 동시성을 데이터 모델로 푼다

이슈 한 개에 대한 동시 작업을 막기 위해, Paperclip은 낙관적 락이나 CRDT가 아닌 **atomic UPDATE**로 푼다. 핵심은 `doc/SPEC-implementation.md` §10.4.1의 atomic checkout contract — 단일 SQL update 를 시도하고, updated row count 가 0이면 `409`를 반환한다는 규칙 — 이지만, 실제 구현은 `expectedStatuses`, assignee/`checkoutRunId` 조건, `executionRunId` 조건, stale-run 인계(adopt) 경로까지 포함한다. **코드 1**이 그 의사코드를 SQL 한 문장으로 압축한다 — `WHERE` 절의 4개 조건이 *"이 슬롯은 비어 있거나 내가 잡고 있던 것이다"*를 한 트랜잭션 안에서 검증한다.

**코드 1. atomic checkout의 의사코드 — 다중 조건 UPDATE 한 번**

```sql
-- 의사코드 (실제 코드는 server/src/services/issues.ts:4659-4809)
UPDATE issues
   SET assignee_agent_id   = $agentId,
       checkout_run_id     = $rid,
       execution_run_id    = $rid,
       status              = 'in_progress',
       started_at          = now()
 WHERE id = $issueId
   AND status IN ($expectedStatuses)               -- 예: backlog/todo/blocked/in_review
   AND (assignee_agent_id IS NULL
        OR (assignee_agent_id = $agentId
            AND (checkout_run_id IS NULL
                 OR checkout_run_id = $rid)))
   AND (execution_run_id IS NULL
        OR execution_run_id = $rid);
-- affected_rows == 1 인 경우만 성공.
-- 실패 시 동일 agent 의 in_progress·null lock 을 자기 run 으로 입양(adopt)하거나,
-- stale checkout_run_id 를 takeover 하는 별도 경로가 이어진다.
```

이로써 **이슈 단일 assignee + atomic checkout** 두 규칙이 합쳐져 "한 이슈 = 한 에이전트가 동시에 잡고 있는 단 한 회차의 run" 이 된다. `checkout_run_id` 와 `execution_run_id` 의 분리는 *오너십 락* 과 *현재 살아 있는 실행 경로* 를 따로 추적하기 위한 것이다.

## 6. 80여 개 테이블 그룹 지도

전체 스키마는 **표 2** 처럼 6개 도메인으로 군집된다.

**표 2. 80여 개 테이블의 도메인 군집**

| 도메인 | 핵심 테이블 | 부속 테이블 (예시) |
|---|---|---|
| **Foundation** | `companies`, Better Auth 테이블(`user`, `session`, `account`, `verification`), `instance_*` | `company_memberships`, `instance_user_roles`, `cli_auth_challenges`, `invites`, `join_requests`, `board_api_keys`, `cloud_upstream_connections`, `cloud_upstream_runs` (#6548 신규 — 마이그레이션 `0089`, local → cloud 회사 동기화) |
| **Agents & runtime** | `agents`, `agent_api_keys`, `heartbeat_runs` | `agent_config_revisions`, `agent_runtime_state`, `agent_task_sessions`, `agent_wakeup_requests`, `environments`, `environment_leases` |
| **Work** | `issues`, `projects`, `goals` | `issue_comments`, `issue_documents`, `document_revisions`, `issue_attachments`, `issue_work_products`, `issue_relations`, `issue_thread_interactions`, `issue_tree_holds`, `issue_labels`, `labels`, `issue_read_states`, `issue_inbox_archives`, `inbox_dismissals`, `project_workspaces`, `project_goals`, `routines`, `workspace_runtime_services`, `execution_workspaces`, `workspace_operations` |
| **Governance** | `approvals`, `approval_comments`, `activity_log` | `principal_permission_grants`, `issue_approvals`, `issue_execution_decisions` |
| **Finance** | `cost_events`, `budget_policies`, `budget_incidents` | (rollups 는 `companies.spent_monthly_cents`, `agents.spent_monthly_cents` 컬럼으로 달림) |
| **Plugins / extension** | `plugins`, `plugin_company_settings`, `plugin_database_namespaces`, `plugin_migrations`, `plugin_state` | `plugin_config`, `plugin_jobs`, `plugin_logs`, `plugin_managed_resources`, `plugin_entities`, `plugin_webhooks`, `feedback_*`, `assets`, `company_secrets` 시리즈, `company_skills`, `company_logos`, `company_user_sidebar_preferences`, `user_sidebar_preferences` |

표 2의 군집은 `server/src/services/`의 파일명·서비스 경계와 대체로 대응된다. 즉, **데이터 도메인이 곧 서비스 도메인**이다. 새 기능을 추가할 때도 이 군집은 영향 범위를 판단하는 기준이 된다 — 예컨대 "정기 작업 기능을 보강한다"는 요구는 *Work* 도메인의 `routines` 보조 테이블 + `services/routines.ts` 묶음에 주로 한정되고, "비용 알림 채널을 추가한다"는 요구는 *Finance* + *Governance* 두 군집을 동시에 건드린다.

## 7. Drizzle 스키마와 마이그레이션 — Drizzle Studio가 보여 주는 모습

Drizzle ORM 의 스키마-퍼스트(schema-first) 워크플로는 다음과 같다. 코드 2 가 Paperclip 에서 스키마 변경의 3단계 안전 게이트를 보여 준다 — 코드(`schema/*.ts`)를 편집하고 SQL 마이그레이션을 자동 생성한 뒤, 명시적으로 적용한다. 운영 적용은 `pnpm db:migrate` 로 명시적이지만, dev/server boot 경로(`server/src/index.ts`)에는 pending migration 자동 적용 조건이 있어 운영 문맥에 따라 동작이 달라진다.

**코드 2. Drizzle 스키마-퍼스트 마이그레이션 사이클**

```bash
# 1) 스키마 코드를 편집
$EDITOR packages/db/src/schema/issues.ts

# 2) SQL 마이그레이션 자동 생성
pnpm db:generate

# 3) 마이그레이션 적용
pnpm db:migrate
```

생성된 마이그레이션은 `packages/db/src/migrations/` 아래에 누적된다(`packages/db/drizzle.config.ts:5`). 그림 5 는 Drizzle 팀이 공식적으로 제공하는 Drizzle Studio 스크린샷으로, 스키마 시각화와 데이터 편집을 같은 도구에서 할 수 있음을 보여 준다(자세한 내용은 [docs/research/06-drizzle-pglite.md](../research/06-drizzle-pglite.md)).

**그림 5. Drizzle Studio (출처: [orm.drizzle.team/drizzle-studio/overview](https://orm.drizzle.team/drizzle-studio/overview))**

![](assets/external/drizzle/drizzle-studio.webp)

## 8. 임베디드 PostgreSQL — 무설정 dev

`DATABASE_URL` 이 비어 있으면 Paperclip 은 자동으로 `~/.paperclip/instances/default/db/` 에 **embedded PostgreSQL** 인스턴스를 띄운다. 이 패키지는 `packages/db/package.json` 에서 `embedded-postgres@^18.1.0-beta.16` 으로 선언되고, 루트 `pnpm-lock.yaml` 과 `patches/embedded-postgres@*.patch` 에서 `18.1.0-beta.16` 으로 고정된다. WASM 이 아니라 플랫폼별 native PostgreSQL 바이너리를 띄우므로 PGlite (WASM) 보다 호환성이 좋다.

그림 6 은 비교 대상 PGlite (ElectricSQL) 의 데모 스크린샷이다. WASM 으로 브라우저에도 실릴 만큼 가벼운 대안이지만, Paperclip 은 데스크톱 dev 의 무게라면 native PostgreSQL 을 선호한다.

**그림 6. PGlite — WASM PostgreSQL (출처: [github.com/electric-sql/pglite](https://github.com/electric-sql/pglite))**

![](assets/external/pglite/screenshot.png)

운영 환경 전환은 `DATABASE_URL` 환경 변수 하나로 끝난다. **코드 3** 은 자주 쓰는 두 가지 외부 PostgreSQL 연결 — 로컬 Docker, Supabase pooler — 의 URL 형태를 보여 준다. 같은 코드/스키마/마이그레이션이 양쪽 모두에서 그대로 동작한다.

**코드 3. `DATABASE_URL` 한 줄로 외부 PostgreSQL 사용**

```bash
# Docker Postgres
DATABASE_URL=postgres://paperclip:paperclip@localhost:5432/paperclip

# Supabase direct / session (5432) — DDL·마이그레이션에 적합
DATABASE_URL=postgres://postgres.[ref]:[pw]@aws-0-[region].pooler.supabase.com:5432/postgres

# Supabase pooled runtime (6543) + 마이그레이션은 5432 직결
DATABASE_URL=postgres://postgres.[ref]:[pw]@aws-0-[region].pooler.supabase.com:6543/postgres
DATABASE_MIGRATION_URL=postgres://postgres.[ref]:[pw]@aws-0-[region].pooler.supabase.com:5432/postgres
```

## 9. 요약

Paperclip의 데이터 모델은 회사 스코프, 단일 assignee, atomic checkout, 비용 귀속, 감사 로그를 중심으로 구성된다. [03-runtime-execution.md](03-runtime-execution.md)는 이 모델 위에서 heartbeat scheduler · atomic checkout · stranded-work 회복 · silent-run watchdog이 어떻게 동작하는지 분석한다.
