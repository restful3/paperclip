# Cross-reference 검토 라운드 1 결정 로그

라운드 1 응답: `2026-05-14-xref-codex-r1.md` (총 substantive 3건, DEFER 0건)

## ACCEPT (3건 — 모두 코드 대조 검증 완료)

### B-1. Claude billing label 의 pre-ledger vs ledger 레이어 구분

- **R03 §5 line 75 (단락 끝)에 한 문장 추가**
- 진실 검증:
  - adapter result label: `"api" | "subscription" | "metered_api"` (`packages/adapters/claude-local/src/server/execute.ts:119`)
  - ledger 정규화: `api → metered_api`, `subscription → subscription_included` (`server/src/services/heartbeat.ts:1275-1285`)
  - BILLING_TYPES enum: `["metered_api", "subscription_included", "subscription_overage", "credits", "fixed", "unknown"]` (`packages/shared/src/constants.ts:484-491`)
- 수정 내용: 단락 끝에 한 문장 — 이 label 들이 adapter result 의 pre-ledger 값이며 heartbeat 서비스가 ledger 기록 시 정규화한다는 사실 명시.
- 우선순위: R03 쪽 수정 (Ch07은 ledger 중심 설명이라 현재 그대로 맞음).

### C-1. Gemini adapter 호출 형태

- **Ch04 §2 표 1 `gemini-local` 행 + R04 §4 일반화 문구**
- 진실 검증:
  - 실제 buildArgs: `["--output-format", "stream-json", "--approval-mode", "yolo", ..., "--prompt", prompt]` (`packages/adapters/gemini-local/src/server/execute.ts:506-518`)
  - command note: "Prompt is passed to Gemini via --prompt for non-interactive execution." (`:448`)
  - `-p` flag 미사용 (Gemini CLI 공식 예시인 `gemini -p`와 다름)
- 수정 내용:
  - Ch04 §2 표 1: `gemini -p …` → `gemini --output-format stream-json --prompt …` (다른 행의 단축 시그니처 양식 유지)
  - R04 §4: "위에서 정리한 헤드리스 플래그 그대로" → "위에서 정리한 헤드리스 계약을 어댑터 형태로 매핑해" (Gemini가 공식 `-p` 와 달리 `--prompt` + `stream-json` 을 사용한다는 점 분리)
- 비고: R04 §3.3 (Gemini CLI 자체 설명) 은 그대로 — 거기서 인용한 `gemini -p` 는 Gemini CLI 공식 docs 의 invocation 양식이고 외부 사실로서 맞다. 충돌은 §4 의 어댑터 일반화 문장과 Ch04 표에서만 발생.

### D-1. Plugin SSE bridge 라인 범위

- **Ch05 §4 SSE blockquote (line 125)**
- 진실 검증:
  - SSE route 선언: `:1363`
  - 헤더 + flushHeaders: `:1388-1397`
  - 스트리밍 fan-out (`res.write('data: …')`): `:1407-1423`
  - 현재 인용 `1373\~1399` 는 헤더 직전까지만 포함하고 실제 stream delivery 를 놓침
- 수정 내용: Ch05 §4 blockquote 의 `:1373\~1399` → `:1363-1423` (`\~` 도 `-` 로 교체)

## REJECT (0건)

## DEFER (0건)

substantive 외 nit (라인 범위 ±2 등) 없음.

## 다음 라운드 (r2)

위 3건 수정 후 R03·R04·Ch04·Ch05 4 파일을 변경 요약과 함께 codex 에게 재리뷰 요청. `===CODEX_FINAL_APPROVAL===` 기대.
