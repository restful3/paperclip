# Paperclip Research Cycle Codex Handout

## 현재 작업

Paperclip 학습 리포트 후속으로 `docs/research/` R01~R10 본문 합의 사이클을 진행 중이다. R01~R04는 `===CODEX_FINAL_APPROVAL===` 완료됐고, R05~R10 및 최종 보고서가 남아 있다.

## 검토 대상 파일들

- `docs/research/01-agent-orchestration-control-plane.md`
- `docs/research/02-model-context-protocol.md`
- `docs/research/03-claude-code-cli.md`
- `docs/research/04-headless-coding-agents.md`
- `docs/research/05-openclaw-agent-companies.md`
- `docs/research/06-adapter-plugin-architecture.md`
- `docs/research/07-sandbox-workspaces.md`
- `docs/research/08-durable-execution-recovery.md`
- `docs/research/09-approval-governance-patterns.md`
- `docs/research/10-llm-cost-budgeting.md`

합의 리뷰 파일은 `docs/reviews/2026-05-14-rNN-{codex,decisions,changes}-rN.md` 패턴을 사용한다.

## 합의된 사실 묶음

### R01 — Agent orchestration / control plane

- Anthropic workflow/agent 구분은 직접 quote가 아니라 paraphrase로 정정했다.
- Temporal Event History quote는 원문과 일치하도록 정정했다.
- LangGraph의 “agent never loses progress...” 표현은 원문에 없어 paraphrase로 정정했다.
- Paperclip budget hard-stop은 비용 추적축과 집행 scope를 분리했다.
- 추적축: `company` / `agent` / `issue` / `project` / `goal` / `heartbeat_run` / `provider` / `model`.
- 집행 scope: `company` / `agent` / `project`.

### R02 — Model Context Protocol

- ChatGPT MCP 도입 specifics 중 InfoQ가 직접 지원하지 않는 `2025-03` 채택 및 `2025-09-10` 일자 주장은 제거했다.
- Streamable HTTP 보안 규범은 `Origin` 검증 `MUST`, localhost/127.0.0.1 바인딩 `SHOULD`로 분리했다.
- Zylos 실제 수치는 2026-03 기준 18,000+ server 색인, 예시로 Glama 18,374 및 MCP.so 18,250+로 정정했다.
- Paperclip MCP server는 어댑터와 직교하는 도메인 통로로 정리했다.

### R03 — Claude Code CLI

- Claude Code 설치 curl 명령은 `curl -fsSL https://claude.ai/install.sh | bash`로 정정했다.
- “이전에 headless mode로 불렸다” 직접 인용은 공식 headless 문서에서 확인되지 않아 제거했다.
- user-invoked slash skill(`/commit`) 및 built-in command는 interactive 전용이고, 일반 skill 자동 로드는 `-p`에서도 동작하되 `--bare`가 hooks/skills/plugins/MCP/auto memory/`CLAUDE.md` 자동 탐색을 끄는 것으로 분리했다.
- `parse.ts`의 세 신호를 분리했다.
  - `extractClaudeRetryNotBefore`: usage limit reset 문구에서 retry 시각 계산.
  - `isClaudeTransientUpstreamError`: rate-limit / 429 / 503 / 529 / overloaded 등 transient upstream 분류.
  - `detectClaudeLoginRequired`: 로그인 요구를 auth-required 신호로 분리.
- Agent SDK 개명 문장은 Agent SDK overview에서 확인되지 않아 삭제했다.

### R04 — Headless coding agents

- Cursor blog(2025-08-07) 인용은 블로그가 직접 지원하는 범위로 축약했다.
- OpenCode Build/Plan 설명은 OpenCode Agents 문서를 별도 출처로 보강했다.
- Paperclip 어댑터 공통 보고는 usage / cost / session / summary 또는 error 중심이며, tool 정보는 보조 정보로 분리했다.
- `pi-local`만 `toolCalls` 배열을 별도 반환하고, `opencode-local`은 `toolErrors`만 분리한다.
- Pi `buildArgs`는 `--tools` / `--session` / `--skill` 뒤에 `extraArgs`를 spread하고, 마지막 positional argument로 `userPrompt`를 추가한다고 명확화했다.

## 합의 운영 관습

- 평가 기준 5개:
  - 사실 정확성: `path:line` 또는 외부 URL quote 기준.
  - 표현 효율성.
  - 한국어 자연성.
  - 마크다운 관습.
  - 인용 완결성.
- DEFER 정책:
  - 라인 ±1~2 nit.
  - 영문 용어 취향.
  - 단어 어색함.
  - 위 항목은 모두 DEFER이며, substantive 사실 오류만 ACCEPT.
- 응답 형식:
  - `### N. <짧은 제목>`
  - `**위치**`
  - `**현재**`
  - `**문제**`
  - `**제안**`
  - 본문에 `[ACCEPT]` / `[REJECT]` / `[DEFER]` 태그 명시.
  - 마지막 줄에 건수 요약.
- 종료 신호:
  - 추가 수정이 없으면 응답 첫 줄에 정확히 `===CODEX_FINAL_APPROVAL===` 한 줄만 출력한다.
  - 자연어 승인은 종료 신호로 인정하지 않는다.

## 미해결·잔존 이견

0건.

## 사용자 지시·범위 제한

- 자율 모드로 진행한다.
- 중간 보고 없이 final-summary까지 일괄 진행한다.
- 사용자에게 묻지 말고 끝까지 진행한다.
- 사용자가 30초마다 진행 강제 체크를 요청했다.
- ScheduleWakeup 60s와 백그라운드 폴링을 병용한다.

## 다음 단계

- R05 `docs/research/05-openclaw-agent-companies.md` 합의 사이클 시작.
- R06~R10 진행.
- 종료 후 research-final-summary 작성.
- `HANDOFF.md` §2 / §3 / §7 갱신.
