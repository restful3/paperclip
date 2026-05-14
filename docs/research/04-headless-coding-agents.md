---
format: report
target: A4
topic: Headless coding agents (Codex, Cursor, Gemini, OpenCode, Pi)
---

# Headless Coding Agents 비교

## 1. 개요

**헤드리스 코딩 에이전트(headless coding agent)** 는 TUI(터미널 UI)를 거치지 않고, 표준 입력으로 프롬프트를 받아 표준 출력으로 이벤트 스트림(보통 JSONL)을 흘려보내는 비대화식(non-interactive) 실행 모드를 갖춘 CLI 도구를 말한다. 동일한 모델·툴 호출 능력을 IDE 사용자뿐 아니라 CI/CD, GitHub Actions, 그리고 Paperclip 같은 control plane이 워커 프로세스로 직접 spawn해 호출할 수 있도록 만든 형태이다. OpenAI는 `codex exec --json`을 "스크립트로 코덱스 출력을 소비하기 위한" 정식 진입점으로 문서화하고([Codex non-interactive docs](https://developers.openai.com/codex/noninteractive)), Cursor는 `cursor-agent -p --output-format stream-json`을 자동화·CI 용도로 명시한다([Cursor headless docs](https://cursor.com/docs/cli/headless)). Gemini CLI도 `gemini -p ... --output-format json`을 동일하게 "headless mode"로 부른다([Gemini headless docs](https://google-gemini.github.io/gemini-cli/docs/cli/headless.html)).

control plane 입장에서 헤드리스 에이전트는 (1) **자식 프로세스로 spawn 가능** (2) **한 프롬프트–한 종료 코드** 로 라이프사이클이 끊어지고 (3) **JSONL 이벤트** 가 표준화돼 있어 토큰·툴호출·세션 ID를 기계적으로 파싱할 수 있다는 세 가지 전제를 만족한다. 이는 Paperclip이 `packages/adapters/*-local`에서 채택한 표준 워커 모델과 정확히 일치한다.

## 2. 비교 표

| 도구 | 제공 주체 | 헤드리스 호출 | MCP 지원 | 라이선스 / 요금 | 출처 |
|---|---|---|---|---|---|
| **Codex CLI** | OpenAI | `codex exec --json` (JSONL 스트림), `--model`, `--sandbox`, `--output-schema` | 내장. `~/.codex/config.toml` 또는 `codex mcp` 서브커맨드로 STDIO/HTTP 서버 등록 | Apache-2.0 (오픈소스 바이너리) + ChatGPT Plus/Pro/Business/Enterprise 구독 또는 OpenAI API 키 | [github.com/openai/codex](https://github.com/openai/codex), [Codex MCP](https://developers.openai.com/codex/mcp), [ChatGPT plan](https://help.openai.com/en/articles/11369540-using-codex-with-your-chatgpt-plan) |
| **Cursor CLI (cursor-agent)** | Anysphere (Cursor) | `cursor-agent -p --output-format stream-json --workspace ...`, `--model`, `--mode`, `--resume` | Pro 이상에서 지원. `~/.cursor/mcp.json`로 등록 | 클로즈드. Pro $20/월부터, Pro+ $60/월, Ultra $200/월. BYOK API 키 옵션 존재 | [Cursor headless](https://cursor.com/docs/cli/headless), [Cursor MCP](https://docs.cursor.com/en/context/mcp), [Cursor pricing](https://cursor.com/pricing) |
| **Gemini CLI** | Google | `gemini -p "..." --output-format json`, `-m gemini-2.5-pro` | 내장. `~/.gemini/settings.json`에 MCP 서버 정의 | Apache-2.0 오픈소스. 무료 티어(Google 계정 로그인) + Vertex AI/Gemini API 키 BYOK | [github.com/google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli), [Gemini headless](https://google-gemini.github.io/gemini-cli/docs/cli/headless.html), [Gemini MCP](https://google-gemini.github.io/gemini-cli/docs/tools/mcp-server.html) |
| **OpenCode** | sst (SST 팀) | `opencode run "..." --format json`, 또는 `opencode serve`로 HTTP 서버 구동 | 내장. `opencode mcp add/list/auth` CLI로 관리 | MIT 오픈소스. 100% BYOK — Anthropic, OpenAI, Google, Bedrock, Groq 등 75개+ 프로바이더 (Models.dev) | [github.com/sst/opencode](https://github.com/sst/opencode), [OpenCode CLI](https://opencode.ai/docs/cli/), [OpenCode MCP](https://opencode.ai/docs/mcp-servers/) |
| **Pi (pi-coding-agent)** | badlogic / earendil-works (Mario Zechner) | `pi -p "..." --mode json`, `--provider`, `--model provider/model`, `--thinking`, `--session`, `--skill` | **내장 미지원**. README는 "No MCP. Build CLI tools with READMEs (see Skills), or build an extension that adds MCP support."라고 명시 | MIT 오픈소스. 풀-BYOK — Anthropic, OpenAI, Gemini, Bedrock, Groq, OpenRouter 등 + Claude Pro/Max·ChatGPT Plus·Copilot 구독 토큰 사용 가능 | [github.com/badlogic/pi-mono](https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/README.md), [npm @mariozechner/pi-coding-agent](https://www.npmjs.com/package/@mariozechner/pi-coding-agent) |

## 3. 도구별 특징

### 3.1 OpenAI Codex CLI

OpenAI가 직접 만드는 공식 코딩 에이전트로, GitHub의 [openai/codex](https://github.com/openai/codex)에서 Rust로 배포된다. 비대화식 호출은 `codex exec` 서브커맨드가 담당하고, `--json`을 켜면 stdout이 JSONL 스트림으로 바뀌어 reasoning-token 사용량까지 모든 이벤트가 기계 가독 형태로 흘러나온다([Codex non-interactive](https://developers.openai.com/codex/noninteractive)). `--sandbox`(기본 read-only, `workspace-write`, `danger-full-access`), `--output-schema`(JSON Schema 강제), `-o/--output-last-message`, `--ephemeral` 같은 자동화-친화 플래그가 풍부하다. **MCP는 1급 시민(first-class)** 으로, `~/.codex/config.toml`에 STDIO/HTTP 서버를 등록하거나 `codex mcp` 서브커맨드로 추가하면 세션 시작 시 자동 기동된다([Codex MCP](https://developers.openai.com/codex/mcp)). 빌링은 ChatGPT Plus/Pro/Business/Enterprise 구독에 포함되며, CI에서는 `CODEX_API_KEY` 환경변수로 OpenAI API 토큰을 별도 청구 경로로 쓸 수 있다([ChatGPT plan billing](https://help.openai.com/en/articles/11369540-using-codex-with-your-chatgpt-plan)).

### 3.2 Cursor CLI (cursor-agent)

Cursor IDE의 에이전트와 동일한 코어를 그대로 터미널로 꺼낸 도구로, `curl https://cursor.com/install -fsSL | bash`로 설치한다([Cursor headless](https://cursor.com/docs/cli/headless)). 헤드리스 호출은 `cursor-agent -p --output-format <text|json|stream-json> --workspace <path>` 형태이고, `--model`, `--mode ask|agent`, `--resume <session>`, `--force`(승인 없이 파일 변경 적용), `--stream-partial-output` 같은 플래그를 가진다. Cursor Agent CLI 는 Cursor 구독의 일부로 editor 외부(터미널·CI) 에서 동일한 Cursor Agent 와 구독 모델을 그대로 호출할 수 있게 해주는 도구로 출시됐다 ([Cursor blog 2025-08-07](https://cursor.com/blog/cli)). MCP는 Pro 이상 플랜에서 지원되고 모델은 Auto/지정 모두 가능, 요금은 토큰-기반 사용량(Auto 기준 cache-read $0.25/M, input $1.25/M, output $6.00/M)으로 산정된다([Cursor pricing](https://cursor.com/pricing)).

### 3.3 Gemini CLI

Google이 [google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli)에서 Apache-2.0으로 공개한 오픈소스 에이전트이다([Google blog](https://blog.google/innovation-and-ai/technology/developers-tools/introducing-gemini-cli-open-source-ai-agent/)). `npm i -g @google/gemini-cli`로 설치하고 `gemini -p "..." --output-format json -m gemini-2.5-pro`로 헤드리스 실행한다. Gemini 3 / 2.5 Pro의 1M 토큰 컨텍스트, Google Search grounding, 파일·셸·웹페치 툴이 내장이며, `~/.gemini/settings.json`의 `mcpServers` 키로 임의 MCP 서버를 추가할 수 있다([Gemini MCP docs](https://google-gemini.github.io/gemini-cli/docs/tools/mcp-server.html)). Google 계정으로 로그인한 무료 티어가 핵심 차별점으로, Vertex AI / Gemini API 키 BYOK도 병행 지원한다.

### 3.4 OpenCode (sst/opencode)

SST 팀(Serverless Stack)이 [sst/opencode](https://github.com/sst/opencode)에서 MIT 라이선스로 만드는 클라이언트–서버 구조의 코딩 에이전트이다. `curl -fsSL https://opencode.ai/install | bash` 또는 `npm i -g opencode-ai@latest`로 설치되며, **TUI · `opencode run` · `opencode serve`(HTTP API) · `opencode web`(웹 UI 헤드리스 서버)** 네 모드를 동시에 지원한다([OpenCode CLI](https://opencode.ai/docs/cli/)). 헤드리스 자동화는 `opencode run "<prompt>" --format json` 이 표준이고, OpenCode 는 모든 도구가 켜진 **Build** 와 파일 쓰기·셸이 `ask` 로 제한된 **Plan** 두 primary agent 를 기본 제공한다 ([OpenCode Agents](https://opencode.ai/docs/agents/)). **완전한 BYOK** 철학으로 Anthropic, OpenAI, Google, Bedrock, Groq, Azure, OpenRouter 등 Models.dev 카탈로그의 75개+ 프로바이더를 한 설정 파일로 묶고, `opencode mcp add/list/auth` 서브커맨드로 MCP 서버 OAuth까지 처리한다([OpenCode MCP](https://opencode.ai/docs/mcp-servers/)).

### 3.5 Pi (pi-local이 가리키는 대상)

**Paperclip의 `pi-local` 어댑터가 가리키는 Pi가 무엇인지가 이 항목에서 가장 모호한 부분** 이라 솔직히 짚어 둔다. 후보는 (a) Inflection의 소비자 챗봇 Pi, (b) Sourcegraph의 Amp/Cody, (c) `@mariozechner/pi-coding-agent` 셋이었다. Paperclip 어댑터 코드(`packages/adapters/pi-local/src/server/execute.ts`, `models.ts`)를 직접 읽으면, 호출 인자가 `pi -p --mode json --append-system-prompt ... --provider ... --model provider/model --thinking ... --tools read,bash,edit,write,grep,find,ls --session <file> --skill <dir>` 형태이고, 모델 디스커버리는 `pi --list-models`를 호출하며 데이터 디렉터리는 `~/.pi/agent/skills` 와 `~/.pi/paperclips`이다. 이 시그니처(특히 `--mode json|rpc`, `--thinking off|minimal|low|medium|high|xhigh` 6단계, `provider/model` 형식, `~/.pi/agent/`)는 Mario Zechner(badlogic)의 [pi-mono / pi-coding-agent](https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/README.md)와 정확히 일치한다([npm @mariozechner/pi-coding-agent](https://www.npmjs.com/package/@mariozechner/pi-coding-agent)).

따라서 `pi-local`이 가리키는 것은 **Inflection Pi도 Sourcegraph도 아니고**, MIT 라이선스 오픈소스 코딩 에이전트인 `@mariozechner/pi-coding-agent`(상위 모노레포 `badlogic/pi-mono`, GitHub org는 `earendil-works/pi`로 미러)이다. 설치는 `curl -fsSL https://pi.dev/install.sh | sh` 또는 `npm i -g @earendil-works/pi-coding-agent`. 헤드리스는 `pi -p --mode json`이고, **MCP는 의도적으로 빼고 "Skills"(README가 붙은 CLI 툴) 기반 확장** 을 채택한 점이 동급 도구들과 가장 큰 차별점이다 — 어댑터가 `~/.pi/agent/skills`에 심볼릭 링크를 심어 Paperclip 표준 스킬을 주입하는 것도 이 때문이다([pi-coding-agent README](https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/README.md)). Anthropic Claude Pro/Max, ChatGPT Plus/Pro, GitHub Copilot 구독 토큰을 직접 사용할 수 있어 BYOK 폭이 매우 넓다.

## 4. Paperclip과의 관계

Paperclip 모노레포의 `packages/adapters/{codex|cursor|gemini|opencode|pi}-local`은 모두 동일한 패턴을 따른다 — control plane(서버)이 task를 push하면 어댑터가 워크스페이스를 동기화한 뒤 위에서 정리한 헤드리스 계약을 어댑터 형태로 매핑해 자식 프로세스를 `spawn`하고 (예: `gemini-local` 은 공식 docs 의 `gemini -p` 예시 대신 `gemini --output-format stream-json ... --prompt <p>` 로 호출 — `packages/adapters/gemini-local/src/server/execute.ts:506-518`), JSONL stdout 을 파싱해 공통적으로 token usage · cost · 세션 ID · 요약 / 오류를 회수하고, 도구별로 가능한 경우 tool 이벤트나 tool error 를 보조 정보로 처리해 control plane 에 보고한다 — 다섯 어댑터 중 `pi-local/src/server/parse.ts:14-31` 만 `toolCalls` 배열을 별도 반환하고, `opencode-local/src/server/parse.ts:26` 은 `toolErrors` 만 분리하며, 나머지는 usage / session / cost 중심이다. 구체적으로 `cursor-local/src/server/execute.ts:574`는 `["-p", "--output-format", "stream-json", "--workspace", cwd, "--resume", sessionId, "--model", model, "--mode", mode, "--yolo"]`를, `pi-local/src/server/execute.ts:618-641` 의 `buildArgs` 는 `--mode json` · `-p` · `--append-system-prompt` 를 항상 넣고, 값이 있을 때만 `--provider` · `--model` · `--thinking` 을 조건부로 추가한 뒤, `--tools` · `--session` · `--skill` 뒤에 `extraArgs` 를 spread 하고 마지막 positional argument 로 `userPrompt` 를 추가한다. 즉 Paperclip은 새 에이전트를 추가할 때 워커 코어를 다시 짜지 않고, 그 도구의 헤드리스 CLI 계약(인자·JSONL 이벤트·세션 파일 경로)만 한 어댑터로 흡수한다 — 헤드리스 CLI 표준화의 가치가 control plane 스케일에서 발생하는 지점이다.

## 5. 출처

- [openai/codex (GitHub)](https://github.com/openai/codex)
- [Codex non-interactive mode](https://developers.openai.com/codex/noninteractive)
- [Codex CLI reference](https://developers.openai.com/codex/cli/reference)
- [Codex MCP docs](https://developers.openai.com/codex/mcp)
- [Using Codex with your ChatGPT plan](https://help.openai.com/en/articles/11369540-using-codex-with-your-chatgpt-plan)
- [Cursor Agent CLI (announcement)](https://cursor.com/blog/cli)
- [Cursor headless CLI docs](https://cursor.com/docs/cli/headless)
- [Cursor MCP docs](https://docs.cursor.com/en/context/mcp)
- [Cursor pricing](https://cursor.com/pricing)
- [google-gemini/gemini-cli (GitHub)](https://github.com/google-gemini/gemini-cli)
- [Gemini CLI headless mode](https://google-gemini.github.io/gemini-cli/docs/cli/headless.html)
- [Gemini CLI MCP servers](https://google-gemini.github.io/gemini-cli/docs/tools/mcp-server.html)
- [Google blog: Introducing Gemini CLI](https://blog.google/innovation-and-ai/technology/developers-tools/introducing-gemini-cli-open-source-ai-agent/)
- [sst/opencode (GitHub)](https://github.com/sst/opencode)
- [OpenCode CLI docs](https://opencode.ai/docs/cli/)
- [OpenCode MCP docs](https://opencode.ai/docs/mcp-servers/)
- [badlogic/pi-mono — pi-coding-agent README](https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/README.md)
- [npm @mariozechner/pi-coding-agent](https://www.npmjs.com/package/@mariozechner/pi-coding-agent)
- [DeepWiki: Pi Model Resolution & Thinking Levels](https://deepwiki.com/badlogic/pi-mono/4.7-model-resolution-and-thinking-levels)
- [Paperclip repository (paperclipai/paperclip)](https://github.com/paperclipai/paperclip)
