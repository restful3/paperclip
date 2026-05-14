# 챕터 ↔ 리서치 cross-reference 정합성 검토 — 라운드 1

## A. R02 §5 ↔ Ch04 §6 — MCP 서버

### 확인된 사실 일치

- R02의 "얇은 MCP wrapper" 설명은 `packages/mcp-server/README.md:5-6` 과 일치한다.
- `McpServer` + `StdioServerTransport` 구성은 `packages/mcp-server/src/index.ts:1-16`, `:26-29` 와 일치한다.
- R02가 예로 든 `paperclipListIssues`, `paperclipCheckoutIssue`, `paperclipUpsertIssueDocument` 는 실제 tool surface 에 존재한다(`packages/mcp-server/src/tools.ts:254`, `:459`, `:520`).
- R02의 Ch04 참조 범위 `docs/report/04-adapters-and-skills.md:145-155` 는 실제로 MCP 서버 호출 통로와 adapter 계약과의 직교 관계를 가리킨다.

### 불일치 / 드리프트 후보

- 없음.

### 확실하지 않은 항목

- 없음.

## B. R03 §3·§5 ↔ Ch04 §1\~§3 + Ch07 — Claude Code CLI / claude-local / cost

### 확인된 사실 일치

- R03의 `buildClaudeArgs` 시작 인자 `["--print", "-", "--output-format", "stream-json", "--verbose"]` 는 실제 코드와 일치한다(`packages/adapters/claude-local/src/server/execute.ts:662-667`). Ch04의 `claude -p --output-format stream-json` 는 같은 실행 모드의 요약 표기다.
- R03의 Claude stream parsing 설명은 `system/init`, `assistant`, `result` 처리 및 usage/cost 추출과 일치한다(`packages/adapters/claude-local/src/server/parse.ts:17-85`).
- R03의 Bedrock/API/subscription 감지 설명은 adapter result 단계에서는 맞다(`packages/adapters/claude-local/src/server/execute.ts:111-123`, `:920-924`).
- Ch04의 현재 adapter 계약 `execute` + `testEnvironment` 필수 설명은 `packages/adapter-utils/src/types.ts:349-352` 와 일치한다.

### 불일치 / 드리프트 후보

#### B-1. Claude billing label 과 cost ledger billing type 의 레이어 구분

- 위치: `docs/research/03-claude-code-cli.md:75` vs `docs/report/07-governance-cost.md:78`, `:101-108`
- 사실 충돌: R03은 Claude adapter가 `api` / `subscription` / `metered_api` billing type 을 분류한다고 쓰지만, Ch07의 cost ledger 문맥에서 실제 `cost_events.billing_type` 은 shared `BILLING_TYPES` 값만 저장한다. 즉 `api` 는 ledger에서 `metered_api` 로, `subscription` 은 `subscription_included` 로 정규화된다.
- 진실 소스: adapter result 값은 `packages/adapters/claude-local/src/server/execute.ts:119-123`, `:920-924`; ledger 정규화는 `server/src/services/heartbeat.ts:1275-1285`, cost event 생성은 `server/src/services/heartbeat.ts:6645-6683`; 허용 ledger 값은 `packages/shared/src/constants.ts:484-491`.
- 제안 수정: R03 §5에 "이 값들은 adapter result 의 pre-ledger label 이며, heartbeat ledger 기록 시 `api`→`metered_api`, `subscription`→`subscription_included` 로 정규화된다"를 한 문장 보강한다. Ch07은 현재 ledger 중심 설명이므로 수정 우선순위는 R03 쪽이다.
- 심각도: substantive (ACCEPT 후보)

### 확실하지 않은 항목

- 없음.

## C. R04 §4 ↔ Ch04 §2 — headless coding adapters

### 확인된 사실 일치

- R04와 Ch04 모두 built-in 런타임 어댑터 목록에 `codex-local`, `cursor-local`, `gemini-local`, `opencode-local`, `pi-local` 을 같은 역할로 둔다.
- Cursor 인자 설명은 실제 `buildArgs` 와 일치한다(`packages/adapters/cursor-local/src/server/execute.ts:580-587`).
- Pi 인자 ordering 설명은 실제 `buildArgs` 와 일치한다(`packages/adapters/pi-local/src/server/execute.ts:618-641`).
- Pi만 `toolCalls` 를 별도 반환하고 OpenCode만 `toolErrors` 를 분리한다는 R04 설명은 `packages/adapters/pi-local/src/server/parse.ts:3-15`, `:31-44` 및 `packages/adapters/opencode-local/src/server/parse.ts:22-31`, `:64-70` 과 일치한다.

### 불일치 / 드리프트 후보

#### C-1. Gemini adapter 호출 형태

- 위치: `docs/research/04-headless-coding-agents.md:51` + `docs/report/04-adapters-and-skills.md:33`
- 사실 충돌: R04 §4는 다섯 local adapter 가 "위에서 정리한 헤드리스 플래그 그대로" spawn 된다고 일반화하고, Ch04 표는 `gemini-local` 호출 형태를 `gemini -p …` 로 적는다. 하지만 Paperclip의 현재 `gemini-local` adapter 는 `-p` 를 쓰지 않고 `["--output-format", "stream-json", ... "--prompt", prompt]` 형태로 실행한다.
- 진실 소스: `packages/adapters/gemini-local/src/server/execute.ts:506-518`; 같은 파일의 command note 도 prompt 전달 방식을 `--prompt` 라고 설명한다(`packages/adapters/gemini-local/src/server/execute.ts:448`).
- 제안 수정: Ch04 표의 `gemini-local` 호출 형태를 `gemini --output-format stream-json --prompt …` 로 고친다. R04 §4의 "위에서 정리한 헤드리스 플래그 그대로"는 "도구별 headless 계약을 Paperclip adapter 형태로 매핑한다"로 약화하고, Gemini는 공식 문서의 `-p` 예시와 달리 현재 adapter가 `--prompt` + `stream-json` 를 쓴다고 분리한다.
- 심각도: substantive (ACCEPT 후보)

### 확실하지 않은 항목

- 없음.

## D. R09 §5 ↔ Ch03 §1·§7 + Ch05 §4 — SSE / heartbeat / WebSocket

### 확인된 사실 일치

- R09의 `wakeup enqueue → run claim → adapter.execute(ctx)` 설명은 Ch03의 heartbeat 시퀀스와 일치한다(`docs/report/03-runtime-execution.md:11-17`, `server/src/services/heartbeat.ts:7647-7663`).
- silent-run watchdog 이 `lastOutputAt` 기준 output silence 를 본다는 R09 설명은 Ch03 §7과 일치한다(`docs/report/03-runtime-execution.md:92-98`, `server/src/services/recovery/service.ts:676-760`, `:1156-1188`).
- WebSocket endpoint `/api/companies/:companyId/events/ws` 와 upgrade 인증 설명은 Ch05 §4 및 코드와 일치한다(`docs/report/05-server-api.md:90-106`, `server/src/realtime/live-events-ws.ts:63-176`).
- `LIVE_EVENT_TYPES` 9개 설명은 shared constants 와 일치한다(`packages/shared/src/constants.ts:596-606`).

### 불일치 / 드리프트 후보

#### D-1. Plugin SSE bridge 라인 범위

- 위치: `docs/research/09-sse-heartbeat-patterns.md:47` vs `docs/report/05-server-api.md:125`
- 사실 충돌: R09는 plugin bridge stream 파일만 인용하고, Ch05는 `server/src/routes/plugins.ts:1373\~1399` 를 SSE bridge 근거로 제시한다. 현재 코드에서 실제 SSE route 선언은 `:1363` 에 시작하고, 이벤트 타입과 `data:` 프레임을 쓰는 핵심 fan-out 콜백은 `:1407-1423` 에 있다. Ch05의 범위는 headers/initial comment까지만 포함하고 실제 stream delivery 구간을 놓친다.
- 진실 소스: `server/src/routes/plugins.ts:1363-1423`.
- 제안 수정: Ch05 §4의 인용 범위를 `server/src/routes/plugins.ts:1363-1423` 으로 고친다. R09 §5도 원하면 `server/src/routes/plugins.ts:1363-1423` 을 붙여 Ch05와 같은 범위로 맞춘다.
- 심각도: substantive (ACCEPT 후보)

### 확실하지 않은 항목

- 없음.

## 요약 표

| 쌍 | substantive 불일치 건수 |
|---|---:|
| A | 0 |
| B | 1 |
| C | 1 |
| D | 1 |

총계 — substantive 3건 / DEFER 0건.
