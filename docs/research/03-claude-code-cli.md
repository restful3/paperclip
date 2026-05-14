---
format: report
target: A4
topic: Anthropic Claude Code (headless agent runtime)
---

# Anthropic Claude Code — Headless Agent Runtime

## 1. 개요

**Claude Code (Anthropic)** 는 코드베이스를 읽고, 파일을 편집하고, 셸 명령을 실행하며, 개발 도구와 통합되는 에이전틱 코딩 도구이다. Anthropic 공식 문서는 이를 "터미널·IDE·데스크톱 앱·브라우저에서 동작하는 AI 코딩 어시스턴트"로 정의하며, 모든 표면이 동일한 엔진을 공유한다고 명시한다([Claude Code overview](https://code.claude.com/docs/en/overview)).

배포 표면은 다음과 같이 정리된다([overview](https://code.claude.com/docs/en/overview)).

- **Terminal CLI** — `curl -fsSL https://claude.ai/install.sh | bash` 또는 Homebrew, WinGet, apt/dnf/apk로 설치하는 풀 기능 CLI.
- **VS Code · JetBrains 플러그인** — 인라인 diff, `@`-mention, plan 검토, 대화 히스토리 통합.
- **Desktop 앱(macOS/Windows)** — 시각적 diff, 다중 세션 사이드바이사이드, 클라우드 세션 트리거.
- **Web (claude.ai/code) · iOS** — 로컬 설치 없이 브라우저/모바일에서 장기 실행 작업 수행.

같은 엔진을 라이브러리로 노출한 것이 **Claude Agent SDK (Python·TypeScript)** 이며, 공식 문서는 "Agent SDK는 Claude Code를 라이브러리로 사용해 프로덕션 AI 에이전트를 빌드"하기 위한 것이라고 설명한다([Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview)). CLI와 SDK는 같은 도구·에이전트 루프·컨텍스트 관리를 공유하며, Anthropic은 "대화형 개발은 CLI, CI/CD와 프로덕션 자동화는 SDK"의 분담을 권장한다(같은 문서).

## 2. 헤드리스 실행 모드

Claude Code의 비대화 실행은 `-p`/`--print` 플래그로 진입한다. 공식 문서는 모든 CLI 옵션이 `-p` 와 함께 동작한다고 설명한다([Run Claude Code programmatically](https://code.claude.com/docs/en/headless)).

| 영역 | 플래그/기능 | 동작 |
|---|---|---|
| 진입 | `-p` / `--print` | 비대화 1회 실행. stdin·stdout 파이프 친화 |
| 출력 형식 | `--output-format text\|json\|stream-json` | `json`은 result·session_id·`total_cost_usd` 포함, `stream-json`은 NDJSON 이벤트 스트림 |
| 토큰 스트림 | `--include-partial-messages` + `--verbose` | `stream_event`로 `text_delta` 단위 전송 |
| 권한 | `--allowedTools "Read,Edit,Bash"` | 도구별 사전 승인. `Bash(git diff *)` 같은 prefix 룰 지원 |
| 권한 모드 | `--permission-mode acceptEdits\|dontAsk\|...` | 세션 전체 기본값. `dontAsk`는 read-only 세트 외에는 모두 거부 |
| 세션 | `--continue`, `--resume <session_id>` | 직전/특정 세션 이어가기 |
| 시스템 프롬프트 | `--append-system-prompt`, `--append-system-prompt-file`, `--system-prompt` | 기본 프롬프트에 추가하거나 통째 교체 |
| MCP | `--mcp-config <file-or-json>` | 외부 MCP 서버 등록 |
| 기타 | `--bare`, `--add-dir`, `--max-turns`, `--model`, `--agents`, `--plugin-dir`, `--dangerously-skip-permissions`, `--json-schema` | 컨텍스트 자동탐색 차단 등 CI 친화 옵션 |

`--bare`는 CI/스크립트 재현성을 위해 hooks·skills·plugins·MCP·`CLAUDE.md` 자동탐색을 모두 끄는 모드이며 "향후 `-p`의 기본값이 될 것"이라고 예고된다([headless](https://code.claude.com/docs/en/headless)). 인증은 `ANTHROPIC_API_KEY` 또는 Bedrock/Vertex/Foundry 변수로 주입한다(같은 문서).

스트리밍 이벤트는 `system/init`(세션 메타·로드된 플러그인·MCP), `assistant`(텍스트 청크), `system/api_retry`(재시도 진행), `result`(최종 usage·`total_cost_usd`)로 구성된다([headless](https://code.claude.com/docs/en/headless)). `/commit` 처럼 사용자가 slash 로 호출하는 **skills 와 built-in commands** 는 인터랙티브 모드 전용이며 `-p` 모드에서는 평문 프롬프트로 기술해야 한다. 반면 일반 skills·hooks·plugins·MCP·auto memory·`CLAUDE.md` 자동 로드 자체는 `-p` 에서도 동작하며, `--bare` 를 지정하면 이 자동 탐색이 모두 꺼진다(같은 문서).

**Hooks** 는 `PreToolUse`, `PostToolUse`, `SessionStart/End`, `UserPromptSubmit`, `Stop`, `PermissionRequest` 등 라이프사이클 이벤트에서 셸·HTTP·MCP·서브에이전트 핸들러를 실행하는 메커니즘이다([Claude Code Hooks](https://code.claude.com/docs/en/hooks)). 설정 위치는 `~/.claude/settings.json`(전역), `.claude/settings.json`(프로젝트 공유), `.claude/settings.local.json`(로컬 전용), 플러그인 `hooks/hooks.json`이다(같은 문서).

## 3. Claude Agent SDK

**Claude Agent SDK** 는 Python(`pip install claude-agent-sdk`)·TypeScript(`npm install @anthropic-ai/claude-agent-sdk`)에서 Claude Code 하니스를 라이브러리로 임베드하는 패키지이다([Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview)). TypeScript SDK는 플랫폼별 Claude Code 네이티브 바이너리를 optional dependency로 번들해 별도 설치가 필요 없다(같은 문서).

핵심 진입점은 `query(prompt, options)`이며 비동기 이터레이터로 `SystemMessage`(`init`), `assistant` 청크, `ResultMessage`를 emit한다(같은 문서). 옵션으로는 다음을 노출한다.

- **Built-in tools** — Read, Write, Edit, Bash, Monitor, Glob, Grep, WebSearch, WebFetch, AskUserQuestion(같은 문서).
- **Hooks** — `PreToolUse`/`PostToolUse`/`SessionStart` 등에 콜백 등록(같은 문서).
- **Subagents (`agents`)** — `AgentDefinition`으로 전문 서브에이전트 선언, `Agent` 도구로 위임(같은 문서).
- **MCP (`mcp_servers`/`mcpServers`)** — Playwright 등 외부 MCP 서버 연결(같은 문서).
- **Permissions** — `allowed_tools`, `permission_mode`로 read-only 등 정책화(같은 문서).
- **Sessions** — `system/init`의 `session_id`를 캡처해 `resume=...`로 재개(같은 문서).
- **Filesystem 설정** — `.claude/skills/*/SKILL.md`, `.claude/commands/*.md`, `CLAUDE.md`, `plugins`를 자동 로드. `setting_sources`/`settingSources`로 로드 범위 제한(같은 문서).

SDK는 또한 **Anthropic Client SDK 와의 차이** 를 강조한다 — 후자는 사용자가 tool loop를 직접 구현해야 하지만, Agent SDK는 Claude가 도구 실행 루프를 자동 처리한다(같은 문서). 그 대척점에는 Anthropic 인프라가 샌드박스를 호스팅하는 **Managed Agents (REST API)** 가 있다(같은 문서).

## 4. 토큰·비용 리포팅 표면

`-p --output-format json` 응답과 `stream-json`의 마지막 `result` 이벤트는 다음을 포함한다([headless](https://code.claude.com/docs/en/headless)).

- `result` — 최종 텍스트(또는 `--json-schema` 사용 시 `structured_output`)
- `session_id` — `--resume`에 다시 넘길 수 있는 식별자
- `usage` — `input_tokens`, `cache_read_input_tokens`, `output_tokens` 등(스키마는 [Paperclip claude-local parser](https://github.com/paperclipai/paperclip/blob/master/packages/adapters/claude-local/src/server/parse.ts)에서도 확인 가능)
- `total_cost_usd` 와 모델별 비용 분해 — "스크립트 호출자가 호출당 지출을 사용량 대시보드 없이 추적할 수 있다"고 명시([headless](https://code.claude.com/docs/en/headless))

스트리밍 중에는 `system/api_retry` 이벤트로 재시도 진행 상황(`attempt`, `max_retries`, `retry_delay_ms`, `error_status`)을 노출하며, `system/init`은 로드된 플러그인 목록·`plugin_errors`를 알려 CI에서 플러그인 누락 시 실패 처리를 가능하게 한다([headless](https://code.claude.com/docs/en/headless)).

## 5. Paperclip과의 관계

**Paperclip** 의 `packages/adapters/claude-local`은 위 헤드리스 모드를 통째로 활용한다. `src/server/execute.ts`의 `buildClaudeArgs`는 정확히 `["--print", "-", "--output-format", "stream-json", "--verbose"]`로 시작해 `--resume`(세션 재개), `--model`, `--effort`, `--max-turns`, `--append-system-prompt-file`, `--add-dir`, 사용자 지정 `extraArgs`를 덧붙인다([execute.ts L655 부근](https://github.com/paperclipai/paperclip/blob/master/packages/adapters/claude-local/src/server/execute.ts)). 프롬프트는 stdin으로 흘려보내고, 어댑터는 NDJSON을 한 줄씩 파싱해 `system/init`에서 `session_id`를, `assistant`에서 텍스트 청크를, 마지막 `result`에서 `usage`(input/cache/output)와 `total_cost_usd`를 추출한다([parse.ts](https://github.com/paperclipai/paperclip/blob/master/packages/adapters/claude-local/src/server/parse.ts)).

Bedrock 인증(`CLAUDE_CODE_USE_BEDROCK=1` 또는 `ANTHROPIC_BEDROCK_BASE_URL`) 감지로 **billing type(`api`/`subscription`/`metered_api`)** 을 분류하고 (`packages/adapters/claude-local/src/server/execute.ts:119-123`), `parse.ts` 는 세 신호를 분리해 처리한다 — (1) usage limit reset 문구는 `extractClaudeRetryNotBefore` 가 매칭해 `retryNotBefore` 시각으로 변환하고, (2) rate-limit / 429 / 503 / 529 / overloaded 등 일시 장애는 `isClaudeTransientUpstreamError` 가 transient upstream 으로 분류하며, (3) 로그인 요구는 `detectClaudeLoginRequired` 가 별도 auth-required 신호로 분리한다 (`packages/adapters/claude-local/src/server/parse.ts:9-15`, `:132-149`, `:355-391`). 즉 Claude Code의 헤드리스 JSON 스트림은 paperclip의 하트비트 워커 사이클과 자연스럽게 맞물려, 각 tick마다 비용·토큰·세션 상태를 정확히 회수할 수 있는 외피로 기능한다. 단, 위 세 label 은 **adapter result 단계의 pre-ledger 값** 이며 — `cost_events` ledger 에는 heartbeat 서비스가 `api → metered_api`, `subscription → subscription_included` 로 정규화해 기록한다 (`server/src/services/heartbeat.ts:1275-1285`; 허용 ledger 값은 `packages/shared/src/constants.ts:484-491`).

## 6. 출처

- [Claude Code overview](https://code.claude.com/docs/en/overview)
- [Run Claude Code programmatically (headless mode)](https://code.claude.com/docs/en/headless)
- [CLI reference](https://code.claude.com/docs/en/cli-reference)
- [Claude Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview)
- [Claude Code Hooks](https://code.claude.com/docs/en/hooks)
- [anthropics/claude-agent-sdk-typescript](https://github.com/anthropics/claude-agent-sdk-typescript)
- [anthropics/claude-agent-sdk-python](https://github.com/anthropics/claude-agent-sdk-python)
- [anthropics/claude-agent-sdk-demos](https://github.com/anthropics/claude-agent-sdk-demos)
- [paperclip claude-local execute.ts](https://github.com/paperclipai/paperclip/blob/master/packages/adapters/claude-local/src/server/execute.ts)
- [paperclip claude-local parse.ts](https://github.com/paperclipai/paperclip/blob/master/packages/adapters/claude-local/src/server/parse.ts)
