---
format: report
target: A4
topic: 설치와 운영 (08)
---

# Usage — 설치 · 회사 만들기 · 첫 운영

## 1. 사전 준비

| 요구 | 버전 | 비고 |
|---|---|---|
| Node.js | ≥ 20 (LTS) | `package.json engines` 강제 |
| pnpm | ≥ 9 | `packageManager: pnpm@9.15.4` |
| OS | macOS · Linux · Windows | embedded-postgres 18 native 바이너리가 깔린다 |
| (옵션) Docker | 24+ | local Docker Postgres 모드용 |
| (옵션) Tailscale | — | private 네트워크 모드용 |

## 2. 5분 안에 시작하기 — `local_trusted` 모드

가장 마찰이 적은 방법은 임베디드 Postgres + 로그인 없는 로컬 모드다. **코드 1**은 4줄로 끝나는 시작 절차다 — Node 20+, pnpm 9+ 외에 추가 설치는 없으며, `pnpm dev`가 임베디드 PostgreSQL · 마이그레이션 · API · UI를 한 origin 위에 띄운다.

**코드 1. local_trusted 모드 — 5분 시작 시퀀스**

```bash
git clone https://github.com/paperclipai/paperclip.git
cd paperclip
pnpm install
pnpm dev
# → http://localhost:3100
```

`pnpm dev`가 만든 부산물은 `~/.paperclip/instances/default/` 아래에 산다. 이 모드에서는 보드 UI가 즉시 회사 생성 화면으로 데려간다. 인증·세션은 없으며, 단일 운영자(`local-board` pseudo-user)가 모든 권한을 가진다.

리셋이 필요하면 `~/.paperclip/instances/default/db/` 디렉터리만 지우고 다시 띄우면 된다.

## 3. CLI 기본 흐름 — `paperclipai`

`pnpm paperclipai` 명령은 `cli/src/index.ts`의 디스패처를 부른다. 자주 쓰는 서브커맨드는 **코드 2**와 같다 — 운영 라이프사이클(`onboard` → `run` → `doctor`)은 위쪽, 부수 작업(`configure` · `plugin install` · `db:backup`)은 아래쪽이다.

**코드 2. paperclipai CLI — 자주 쓰는 서브커맨드**

```bash
pnpm paperclipai onboard          # 인터랙티브 초기 설정 (모드/바인드/secrets)
pnpm paperclipai onboard --yes    # 기본값으로 즉시 진행 (local_trusted/loopback)
pnpm paperclipai run              # dev가 아닌 한 번 부팅
pnpm paperclipai run --bind lan   # 첫 실행(설정 없음) 때 authenticated/private + LAN 바인드
pnpm paperclipai doctor           # 환경 점검
pnpm paperclipai configure --section secrets  # 비밀 관리 설정
pnpm paperclipai plugin install <pkg|path>    # 어댑터 플러그인 설치
pnpm paperclipai db:backup        # 논리 덤프
```

`onboard`가 묻는 첫 질문은 reachability다(자세한 내용 `doc/DEPLOYMENT-MODES.md` §4). **코드 3**은 4가지 reachability 옵션이며, 1번이 *나만 쓴다*, 2\~3번이 *팀이 쓴다*, 4번이 *외부에 공개한다*는 운영 톤에 대응한다.

**코드 3. onboard reachability 4가지 옵션**

```text
1. Trusted local       → bind=loopback, mode=local_trusted/private
2. Private network     → bind=lan,      mode=authenticated/private
3. Tailnet             → bind=tailnet,  mode=authenticated/private
4. Custom              → 직접 입력 + authenticated/public 도 가능
```

## 4. 보드 UI 첫 화면 — 회사를 만들자

UI가 처음 띄우는 흐름은 두 갈래다.

(A) **새 회사 만들기**
- `Company name` — 첫 화면 필수 입력
- `Mission / goal` — 선택, 비워 둘 수 있음 (e.g., *"Build the #1 AI note-taking app to $1M MRR."*)

첫 화면은 이 두 입력으로 끝난다(`ui/src/components/OnboardingWizard.tsx:681,700`). `description` · `brand_color` · `attachment_max_bytes` 같은 항목은 회사 설정(`/company/settings`)에서, **월 예산은** 별도의 예산 API(`PATCH /api/companies/:companyId/budgets` — route-local 정의 `server/src/routes/costs.ts:280`, `/api` mount `server/src/app.ts:196-224, 314-317`)와 비용 화면(`/costs`)에서 조정한다. `issue_prefix` 는 서버가 회사명에서 자동 파생한다(`server/src/services/companies.ts:117-150`).

(B) **기존 회사 불러오기 (import)**
- `/company/import` 페이지에서 회사 패키지 (Agent Companies spec/v1 호환) 업로드
- 템플릿 모드(구조만) 또는 스냅샷 모드(상태 포함) 선택

회사가 만들어지면 사이드바에 회사 항목이 추가되고, 모든 후속 작업은 그 회사 컨텍스트 안에서 일어난다.

## 5. 첫 에이전트 — CEO 부터

`/agents/new` 페이지가 다음을 묻는다.

| 입력 | 예시 | 설명 |
|---|---|---|
| name | "Aurora" (첫 에이전트는 자동 `CEO`) | 사람 친화 이름 |
| title | CEO | 표시용 |
| role | `ceo` (첫 에이전트 자동) / `engineer` / `marketer` … | 첫 에이전트는 자동으로 ceo, 이후 자유 선택 |
| reports_to | (첫 에이전트는 없음 — picker disabled) | 두 번째부터 picker 활성화. 현재 구현은 `No manager` 선택도 허용하므로 *CEO-only root* 는 운영 규칙·향후 invariant 로 봐야 한다 (`ui/src/components/ReportsToPicker.tsx:79-90`, validator `packages/shared/src/validators/agent.ts:66-79`). |
| **adapter_type** | `claude_local` | 어떤 런타임으로 깰까 |
| **adapter_config** | jsonb | 어댑터마다 다른 모양 (Claude Code 면 SKILL/CLAUDE.md, system prompt, model, max_turns 등) |
| **runtime_config** | jsonb | heartbeat · interval · cheap model 등 (`buildNewAgentRuntimeConfig`) |
| desiredSkills | (선택) | 회사 라이브러리에서 골라 미리 부여 |

생성 화면의 핵심 identity/payload 항목은 위와 같지만, `AgentConfigForm` 안에서 default environment, adapter-specific config, run policy도 함께 받는다(`ui/src/components/AgentConfigForm.tsx:783-1193`, `ui/src/lib/new-agent-hire-payload.ts:23-39`). `icon` · `capabilities` · 월 예산 같은 항목은 생성 직후 에이전트 설정(edit) 화면에서 채운다 — payload 는 `budgetMonthlyCents: 0` 으로 고정해 hire 한다.

`require_board_approval_for_new_agents=true`라면 이 시점에 `approvals` row가 만들어지고 `agents.status='pending_approval'`로 머문다. 보드가 승인해야 한다.

후속 임원/에이전트도 현재 구현에서는 명시적인 agent hire 흐름으로 생성된다. `require_board_approval_for_new_agents=true` 이면 각 hire 가 `hire_agent` approval 로 들어가고, 보드 승인 후 활성화된다. 그림 8은 한 회사의 org tree 예시 — CEO 아래 4명의 임원, 그 아래 엔지니어/마케터/지원 — 를 보여 준다.

**그림 8. 한 회사의 org tree 예시 — CEO 아래 CTO/CMO/CFO/COO 와 그 하위 보고 라인**

![](figs/fig-08-orgchart.svg)

그림 8의 *깊이*는 두 가지 운영 시그널을 제공한다. 첫째, 운영 원칙으로는 CEO 중심 위임을 택할 수 있지만, 현재 보드 UI/API 는 전체 org tree 의 에이전트를 직접 조회·생성·수정할 수 있다(`ui/src/App.tsx:87-96`, `server/src/routes/agents.ts:571-580`, `:2540-2548`, `:677-685`). 따라서 깊은 트리는 *보드가 직접 봐야 할 표면* 보다 *책임 추적 표면* 을 넓힌다. 둘째, 트리 깊이가 늘어날수록 *책임 추적의 비용*도 늘어난다. 초기 에이전트 회사는 *2층(CEO + 임원)* 구조가 가장 단순하며, 깊이 3층 이상의 트리는 도메인이 정확히 분해된 회사에서 효과적이다. 반대로 깊이가 너무 얕으면 CEO 한 명이 모든 작업을 직접 잡게 되어 atomic checkout으로 인한 직렬화가 병목이 된다.

## 6. 첫 골과 첫 이슈

`/projects` 또는 `/goals`에서 회사 골을 분해한다. SPEC §5의 골 위계는 *Initiative → Project → Milestone → Issue → Sub-issue*다. 실용적으로는 다음 패턴이 유효하다.

1. 보드/사용자가 `/projects` 또는 `/goals` 에서 골을 만들고, `/issues` 에서 실행 이슈를 만든다.
2. 에이전트는 승인된 고용 후 task 를 배정받거나 이슈 생성을 요청/수행할 수 있다.
3. CEO 초기 전략 분해 approval(`approve_ceo_strategy`)은 SPEC 목표로 남아 있다 — 현재 코드는 approval type 상수만 정의한다.
4. 자식 에이전트가 자기 차례에 일을 하고, 비용을 보고하고, 결과를 `in_review` 로 핸드오프한다.

이슈 ID는 `companies.issue_prefix` + `issue_counter`로 회사별 단조 증가 — 예: `PAP-1`, `PAP-2`, ...

## 7. 운영 중 흔한 점검 명령

**코드 4**는 일상 운영의 5가지 점검 묶음이다 — 회사 상태 → 러너 인스턴스 → 마이그레이션 → 전체 검증 → E2E의 사다리. 위에서 아래로 갈수록 무거운 검증이며, 일상은 1\~2번까지로 충분하고 PR 직전에는 4번까지 돌린다.

**코드 4. 일상 점검 5묶음 — health · runner · migration · validation · E2E**

```bash
# 1) 회사 상태
curl -s http://localhost:3100/api/health
curl -s http://localhost:3100/api/companies

# 2) dev 러너 인스턴스 관리
pnpm dev:list
pnpm dev:stop

# 3) 마이그레이션 수동 적용
pnpm db:migrate

# 4) 전체 검증 (PR 전 단계)
pnpm -r typecheck
pnpm test:run
pnpm build

# 5) E2E (UI 흐름이 바뀌었을 때만)
pnpm test:e2e
pnpm test:release-smoke
```

`AGENTS.md` §7의 권고는 *"PR 핸드오프 전에는 전체 체크, 그 외에는 가장 좁은 검증"*이다.

## 8. Docker 한 줄로 띄우기 (옵션)

**코드 5**는 두 가지 Docker 사용 패턴이다 — `docker compose` 로 **PostgreSQL 17 + Paperclip server 풀스택**을 한 명령으로 띄우거나, `docker build` + `docker run` 으로 호스트 DB(또는 임베디드 Postgres) 위에 Paperclip 컨테이너만 묶거나. 후자가 *완전 portable*이라 운영 이전이 가장 단순.

**코드 5. Docker 사용 패턴 — full-stack compose vs Paperclip 단독 컨테이너**

```bash
# 풀스택 (PostgreSQL 17 + Paperclip server). compose 가 둘 다 띄운다
BETTER_AUTH_SECRET=$(openssl rand -hex 32) \
  docker compose -f docker/docker-compose.yml up --build

# 또는 Paperclip 컨테이너만 (임베디드 Postgres 모드)
docker build -t paperclip:dev .
docker run -p 3100:3100 \
  -e BETTER_AUTH_SECRET=$(openssl rand -hex 32) \
  -v paperclip-data:/paperclip paperclip:dev
```

위 compose 는 `db`(postgres:17-alpine) 와 `server`(Paperclip Dockerfile) 두 서비스를 함께 띄우며 `BETTER_AUTH_SECRET` 이 강제된다(`docker/docker-compose.yml:2-16, 18-31`, `doc/DOCKER.md:84-89`). 단독 컨테이너 경로도 마찬가지로 `BETTER_AUTH_SECRET` 이 필요한데, 루트 `Dockerfile` 의 기본값이 `PAPERCLIP_DEPLOYMENT_MODE=authenticated`(`Dockerfile:79-80`)이기 때문이다. 단독 컨테이너는 임베디드 Postgres 모드를 그대로 쓰므로 `/paperclip` 볼륨만 보존하면 된다(`doc/DOCKER.md`).

## 9. 호스팅 시나리오 — Supabase + Vercel

규모가 커지면 호스티드로 옮긴다(`doc/DATABASE.md` §3). **코드 6** 은 Supabase + Vercel 같은 매니지드 환경으로 옮길 때 필요한 핵심 환경 변수다 — DB/마이그레이션·secrets·auth 세 묶음으로 나뉜다.

**코드 6. 호스팅 시나리오 핵심 환경 변수 (Supabase + Vercel 예시)**

```bash
# DB / 마이그레이션
DATABASE_URL=postgres://postgres.[ref]:[pw]@aws-0-[region].pooler.supabase.com:5432/postgres
DATABASE_MIGRATION_URL=...   # 풀러 URL 을 쓰는 경우 직결 URL 을 별도로

# secrets
PAPERCLIP_SECRETS_MASTER_KEY=...   # base64 32바이트, local_encrypted 마스터 키

# authenticated/public 모드 필수 (기본값은 local_trusted + private)
PAPERCLIP_DEPLOYMENT_MODE=authenticated
PAPERCLIP_DEPLOYMENT_EXPOSURE=public
BETTER_AUTH_SECRET=...               # openssl rand -hex 32
PAPERCLIP_PUBLIC_URL=https://your-host  # auth.publicBaseUrl 과 동등, 명시적 base URL
```

`authenticated/public` 모드에서는 Better Auth 세션이 강제되며 `BETTER_AUTH_SECRET` 과 `auth.publicBaseUrl` 이 *함께* 필수다(`cli/src/checks/deployment-auth-check.ts:28-60`). env-only 호스팅을 만들려면 `PAPERCLIP_DEPLOYMENT_MODE` / `EXPOSURE` 도 명시해야 한다 — 미설정이면 `local_trusted` + private 으로 떨어진다(`server/src/config.ts:160-180`).

`local_trusted` 로 오래 사용하다 `authenticated` 로 전환하는 경우 — instance admin 이 `local-board` 한 명뿐일 때에 한해 — 부팅 시 1회용 board-claim URL 이 경고로 표시되어, 로그인한 사용자가 instance admin 으로 승격되고 `local-board` 권한이 강등된다(`doc/DEPLOYMENT-MODES.md:115-126`, `server/src/board-claim.ts:44-73`, `server/src/routes/access.ts:2408-2453`).

## 10. 초기 운영 체크포인트

- **예산 한도** 는 보수적으로 설정하고 `unlimited` 사용을 제한한다. 초기 폭주 비용이 가장 흔한 운영 리스크다.
- **승인 게이트** (`require_board_approval_for_new_agents=true`)를 켜 두면 CEO가 만들려는 에이전트를 보드가 먼저 검토할 수 있다.
- **`/dashboard/live`** 는 `/api/companies/:companyId/live-runs` 조회와 run log polling 으로 *활성/최근 에이전트 run* 을 보여 준다 (server route `server/src/routes/agents.ts:3095-3106`, UI API base `ui/src/api/client.ts:1, 22`, `ui/src/components/ActiveAgentsPanel.tsx:78-81, 109-116`, `ui/src/api/heartbeats.ts:108-120`, `ui/src/components/transcript/useLiveRunTranscripts.ts:250-259`). WebSocket transcript streaming 은 `useLiveRunTranscripts` 의 realtime 옵션이 켜진 화면에서만 사용된다. 회사 *활동 로그* (`/activity`) 와는 다른 화면이다.
- **`/approvals`** 큐와 **활동 로그** 는 일상 점검의 최소 표면이다.

[09-pros-cons.md](09-pros-cons.md)는 이 모든 메커니즘의 *강점·약점·트레이드오프*를 정리한다.
