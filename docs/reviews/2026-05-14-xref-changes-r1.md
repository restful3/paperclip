# Cross-reference 라운드 1 → 2 변경 요약

총 3 파일 수정. 모든 변경은 라운드 1 ACCEPT 항목에 직접 추적됨.

## 1. `docs/research/03-claude-code-cli.md` §5 — billing label 레이어 구분 추가

**위치**: line 75 (단락 끝부분)

**변경 내용**:
- adapter result label 인용에 코드 위치 추가: ``(`packages/adapters/claude-local/src/server/execute.ts:119-123`)``
- 단락 끝에 한 문장 추가 — 위 세 label 이 adapter result *pre-ledger* 값이며, heartbeat 서비스가 `cost_events` ledger 에 `api → metered_api`, `subscription → subscription_included` 로 정규화해 기록한다는 사실.
- 신규 인용: ``(`server/src/services/heartbeat.ts:1275-1285`; 허용 ledger 값은 `packages/shared/src/constants.ts:484-491`)``

**검증 근거** (라운드 1 B-1):
- `execute.ts:119` `resolveClaudeBillingType` 반환 타입 `"api" | "subscription" | "metered_api"` 확인
- `heartbeat.ts:1275-1285` `normalizeLedgerBillingType` switch 매핑 확인
- `constants.ts:484-491` `BILLING_TYPES` enum 6 값 확인 (ledger 에 `api`/`subscription` 자체는 존재하지 않음)

## 2. `docs/report/04-adapters-and-skills.md` 표 1 — gemini-local 호출 형태

**위치**: line 33 (표 1 `gemini-local` 행의 "호출 형태" 셀)

**변경 내용**:
- AS-IS: `gemini -p …`
- TO-BE: `gemini --output-format stream-json --prompt …`

**검증 근거** (라운드 1 C-1):
- `packages/adapters/gemini-local/src/server/execute.ts:506-518` `buildArgs` 실제 인자: `["--output-format", "stream-json", ... "--approval-mode", "yolo", ..., "--prompt", prompt]` 확인
- `-p` flag 미사용 확인 — Gemini CLI 공식 docs 의 `gemini -p "..."` 양식과 Paperclip adapter 양식이 다름

## 3. `docs/research/04-headless-coding-agents.md` §4 — 일반화 문구 약화 + Gemini 분기 명시

**위치**: line 51 (한 문단)

**변경 내용**:
- AS-IS: "위에서 정리한 헤드리스 플래그 그대로 자식 프로세스를 `spawn`하고"
- TO-BE: "위에서 정리한 헤드리스 계약을 어댑터 형태로 매핑해 자식 프로세스를 `spawn`하고 (예: `gemini-local` 은 공식 docs 의 `gemini -p` 예시 대신 `gemini --output-format stream-json ... --prompt <p>` 로 호출 — `packages/adapters/gemini-local/src/server/execute.ts:506-518`), ..."

**검증 근거** (라운드 1 C-1): 동일 — Gemini adapter 가 §3.3 의 공식 invocation 양식과 다름.

**참고**: R04 §3.3 (Gemini CLI 자체 설명) 의 `gemini -p "..."` 인용은 그대로 유지 — 그것은 Gemini CLI 공식 docs 의 외부 사실로서 맞다. 분기는 §4 의 어댑터 일반화 문장에서만 발생.

## 4. `docs/report/05-server-api.md` §4 — SSE bridge 라인 범위 정정

**위치**: line 125 (SSE blockquote)

**변경 내용**:
- AS-IS: ``(`server/src/routes/plugins.ts:1373\~1399`)``
- TO-BE: ``(`server/src/routes/plugins.ts:1363-1423`)``
- 부수적: `\~` → `-` 로 표기 통일

**검증 근거** (라운드 1 D-1):
- `server/src/routes/plugins.ts:1363` SSE route 선언 시작 (`router.get("/plugins/:pluginId/bridge/stream/:channel", ...)`)
- `:1388-1397` 헤더 + `flushHeaders()`
- `:1407-1423` 실제 stream fan-out (`res.write('data: ' + JSON.stringify(event) + '\n\n')`)
- 기존 인용 `1373-1399` 는 헤더 직전에서 시작해 stream delivery 직전에 끊김 — substantive 부정확

## 일관성 점검

위 4 변경 외 본문은 손대지 않음. 라운드 1 에서 0건이었던 쌍 A (R02↔Ch04) 는 변경 없음.

라운드 2 재리뷰 요청은 변경된 4 파일 (`docs/research/03-claude-code-cli.md`, `docs/report/04-adapters-and-skills.md`, `docs/research/04-headless-coding-agents.md`, `docs/report/05-server-api.md`) 의 해당 단락만 다시 보면 충분.
