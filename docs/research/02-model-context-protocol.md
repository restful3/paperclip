---
format: report
target: A4
topic: Model Context Protocol (MCP)
---

# Model Context Protocol (MCP)

## 1. 개요

**Model Context Protocol (MCP)** 은 AI 애플리케이션을 외부 데이터 소스, 도구, 워크플로에 연결하기 위한 오픈 표준이다. 공식 문서는 MCP 를 "AI 애플리케이션을 위한 USB-C 포트"에 비유하며, 표준화된 방식으로 LLM 호스트와 외부 시스템을 연결하는 프로토콜로 정의한다 ([modelcontextprotocol.io introduction](https://modelcontextprotocol.io/introduction)).

MCP 는 **Anthropic** 이 2024년 11월 25일 오픈소스로 공개했다. Anthropic 은 발표문에서 "가장 정교한 모델조차 데이터로부터의 고립에 의해 제약되어 있다 — 정보 사일로와 레거시 시스템 뒤에 갇혀 있다"고 진단하며, 데이터 소스마다 별도의 커넥터를 유지하는 방식이 지속 불가능하다고 지적했다. MCP 는 이 분절된 통합을 "단일 프로토콜"로 대체한다는 것이 핵심 동기다 ([Anthropic news, 2024-11-25](https://www.anthropic.com/news/model-context-protocol)). 초기 도입사로 **Block**, **Apollo**, **Zed**, **Replit**, **Codeium**, **Sourcegraph** 가 거론되었고, 이후 **OpenAI** 도 자사 제품에 MCP 를 채택해 ChatGPT 개발자 모드(beta) 에서 읽기/쓰기 MCP 클라이언트를 full 지원으로 제공한다 ([InfoQ, 2025-10-13](https://www.infoq.com/news/2025/10/chat-gpt-mcp/)).

## 2. 아키텍처와 핵심 개념

MCP 는 client-server 아키텍처 위에 **JSON-RPC 2.0** 메시지 교환을 얹은 stateful 프로토콜이다. 데이터 레이어가 lifecycle, primitives, notifications 를 정의하고 transport 레이어가 채널을 추상화한다 ([Architecture overview](https://modelcontextprotocol.io/docs/learn/architecture)).

| 구성요소 | 역할 |
|---|---|
| **Host** | AI 애플리케이션 본체. 여러 client 를 조율한다 (예: Claude Code, Claude Desktop, VS Code, Cursor). |
| **Client** | host 안에서 server 하나당 하나씩 인스턴스화되어 전용 연결을 유지한다. |
| **Server** | context 와 도구를 제공하는 프로그램. 로컬(stdio) 또는 원격(HTTP)에서 실행 가능. |
| **Tool** | AI 가 호출 가능한 실행 함수. `tools/list` 로 발견하고 `tools/call` 로 실행한다. |
| **Resource** | 파일, DB 레코드, API 응답 등 컨텍스트 데이터. `resources/list` `resources/read` 로 접근. |
| **Prompt** | 시스템 프롬프트나 few-shot 예시 같은 재사용 가능한 상호작용 템플릿. |
| **Sampling** | server 가 host LLM 으로부터 completion 을 요청하는 client primitive. `sampling/createMessage` 사용. |
| **Elicitation** | server 가 사용자에게 추가 정보·확인을 요청. `elicitation/create`. |
| **Notifications** | `tools/list_changed` 같은 단방향 갱신 알림 (응답 없음). |

server 는 tool/resource/prompt 세 가지 primitive 를 노출하고, client 는 sampling/elicitation/logging 을 제공한다. 초기화 시 `initialize` 핸드셰이크로 protocolVersion 과 capabilities 를 협상한다 ([같은 문서](https://modelcontextprotocol.io/docs/learn/architecture)).

## 3. 전송 방식

MCP 명세는 두 가지 표준 transport 를 정의하며, 클라이언트는 가능한 한 stdio 를 지원해야 한다 ([Transports spec](https://modelcontextprotocol.io/docs/concepts/transports)).

| Transport | 배치 | 메시지 흐름 | 특징 |
|---|---|---|---|
| **stdio** | 로컬 (host 가 server 를 subprocess 로 실행) | stdin/stdout 에 newline-delimited JSON-RPC | 네트워크 오버헤드 없음, 단일 client. stderr 는 로깅용. |
| **Streamable HTTP** | 원격 또는 로컬 데몬 | 단일 MCP endpoint 에 HTTP POST + 선택적 SSE 스트림 | 다중 client, OAuth/Bearer 인증, `Mcp-Session-Id` 세션 관리, `Last-Event-ID` 로 재연결 가능. |
| **HTTP+SSE (구)** | 원격 | 별도 GET(SSE) + POST endpoint | 2024-11-05 명세에서 정의, **Streamable HTTP 로 대체되어 deprecated**. backwards-compat 를 위해 일부 server 가 병행 운영. |

Streamable HTTP 는 protocol version `2025-03-26`/`2025-06-18` 에서 자리잡은 현 권장 원격 transport 이며, DNS rebinding 방지를 위해 명세는 server 가 `Origin` 헤더를 검증할 것을 `MUST` 로 요구하고, 로컬 실행 시 `localhost` / `127.0.0.1` 에만 바인딩할 것을 `SHOULD` 로 권고한다 ([같은 문서](https://modelcontextprotocol.io/docs/concepts/transports)).

## 4. 생태계 현황

**SDK** 는 TypeScript 와 Python 이 1차 시민이며, Node·Bun·Deno 를 지원한다. TypeScript SDK 는 `@modelcontextprotocol/sdk` 로 server/client 양쪽 라이브러리를 제공한다 ([typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk)). 공식 reference server 는 filesystem, git, fetch, memory, sequential-thinking, time, everything 등 7종이며 npx 또는 uvx 로 즉시 실행된다 ([modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers)).

**클라이언트** 측에서는 **Claude Code**, **Claude Desktop**, **Cursor**, **VS Code Copilot Chat**, **ChatGPT** 가 모두 MCP 를 지원한다. Cursor 는 `~/.cursor/mcp.json` (전역) 또는 프로젝트 `.cursor/mcp.json` 에 `mcpServers` 객체를 두고 stdio·SSE·Streamable HTTP 세 transport 를 선택할 수 있다 ([Cursor docs](https://cursor.com/docs/context/mcp)). Claude Code 는 `claude mcp add` CLI 로 local·project·user 스코프 단위로 server 를 등록한다 ([Claude Code MCP docs](https://code.claude.com/docs/en/mcp)).

**레지스트리** 는 공식 **Official MCP Registry** (registry.modelcontextprotocol.io) 가 발견 허브 역할을 하며, 커뮤니티 운영 디렉토리 **mcp.so** 도 같이 검색에 사용된다 ([Official MCP Registry](https://registry.modelcontextprotocol.io/)). 외부 분석에 따르면 2026년 3월 기준 커뮤니티 레지스트리는 18,000+ MCP server 를 색인하며 (예: Glama.ai 18,374 · MCP.so 18,250+) 빠르게 성장하고 있다 ([Zylos research, 2026-03-08](https://zylos.ai/research/2026-03-08-mcp-remote-evolution-streamable-http-enterprise-adoption)). 워크플로 자동화 도구 **n8n** 도 *MCP Server Trigger* 노드와 *MCP Client Tool* 노드를 내장해, 워크플로를 MCP server 로 노출하거나 외부 server 를 AI Agent 안에서 소비할 수 있다 ([n8n MCP docs](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-langchain.mcptrigger/)).

## 5. Paperclip과의 관계

Paperclip 의 `packages/mcp-server` 는 자체 비즈니스 로직을 재구현하지 않고 **기존 Paperclip REST API (`/api`) 를 감싸는 얇은 MCP wrapper** 다. `@modelcontextprotocol/sdk` 의 `McpServer` + `StdioServerTransport` 위에서 `paperclipListIssues`, `paperclipCheckoutIssue`, `paperclipUpsertIssueDocument` 같은 tool 들을 등록하고, 환경 변수 `PAPERCLIP_API_URL` / `PAPERCLIP_API_KEY` / `PAPERCLIP_AGENT_ID` 로 받은 자격증명으로 HTTP 호출을 위임한다 (`packages/mcp-server/src/index.ts`, `README.md`). 즉 데이터 흐름은 **agent (Claude Code/Codex/Cursor) → MCP client → Paperclip MCP server (stdio) → Paperclip REST API** 로 단방향 매핑된다.

이 구조 덕분에 Paperclip 은 새 MCP 호환 호스트/에이전트가 등장할 때마다 별도 어댑터를 추가할 필요 없이, 해당 호스트가 Paperclip MCP server 를 표준 방식으로 등록만 하면 어댑터 종류와 무관하게 task·comment·document·approval 도구를 호출할 수 있다. 즉 어댑터 실행 계약(`execute` / `testEnvironment`) 은 *프로세스 차원* 의 통로이고, MCP 는 *도메인 차원* 의 통로로 직교한다 (Ch04 `docs/report/04-adapters-and-skills.md:145-155` 참조). Paperclip 이 control plane 으로서 "회사" 메타데이터를 관리하고, MCP 가 "회사에 출근하는 에이전트들의 표준 인터페이스" 역할을 맡는 분리 구조라고 볼 수 있다.

## 6. 출처

- [Model Context Protocol — Introduction](https://modelcontextprotocol.io/introduction)
- [Anthropic — Introducing the Model Context Protocol (2024-11-25)](https://www.anthropic.com/news/model-context-protocol)
- [MCP Architecture overview](https://modelcontextprotocol.io/docs/learn/architecture)
- [MCP Specification — Transports](https://modelcontextprotocol.io/docs/concepts/transports)
- [modelcontextprotocol/servers (reference servers)](https://github.com/modelcontextprotocol/servers)
- [modelcontextprotocol/typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk)
- [Cursor — Model Context Protocol docs](https://cursor.com/docs/context/mcp)
- [Claude Code — Connect tools via MCP](https://code.claude.com/docs/en/mcp)
- [Official MCP Registry](https://registry.modelcontextprotocol.io/)
- [n8n — MCP Server Trigger node](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-langchain.mcptrigger/)
- [InfoQ — OpenAI Adds Full MCP Support to ChatGPT Developer Mode (2025-10)](https://www.infoq.com/news/2025/10/chat-gpt-mcp/)
- [Zylos Research — MCP's Remote Revolution (2026-03)](https://zylos.ai/research/2026-03-08-mcp-remote-evolution-streamable-http-enterprise-adoption)
