# External Image Manifest

이 디렉터리는 Paperclip 리서치 노트(`docs/research/01-…10-*.md`)가 인용한 공식 출처에서
직접 받은 다이어그램·스크린샷을 담는다. 모든 이미지의 저작권은 원 게시자에 귀속되며,
보고서에서 인용 시 아래 "Source page" 와 "Original URL" 을 함께 표기한다.

## R01 — AI Agent Orchestration / Control Plane

- `assets/external/building-effective-agents/augmented-llm.png`
  - Source page: https://www.anthropic.com/research/building-effective-agents
  - Original URL: https://www-cdn.anthropic.com/images/4zrzovbb/website/d3083d3f40bb2b6f477901cc9a240738d3dd1371-2401x1000.png
  - Caption: The "augmented LLM" building block (LLM + retrieval + tools + memory)
  - License/attribution: Anthropic — "Building Effective Agents" (Schluntz & Zhang, Anthropic Engineering)

- `assets/external/building-effective-agents/orchestrator-workers.png`
  - Source page: https://www.anthropic.com/research/building-effective-agents
  - Original URL: https://www-cdn.anthropic.com/images/4zrzovbb/website/8985fc683fae4780fb34eab1365ab78c7e51bc8e-2401x1000.png
  - Caption: Orchestrator-workers workflow pattern
  - License/attribution: Anthropic — "Building Effective Agents"

- `assets/external/building-effective-agents/evaluator-optimizer.png`
  - Source page: https://www.anthropic.com/research/building-effective-agents
  - Original URL: https://www-cdn.anthropic.com/images/4zrzovbb/website/14f51e6406ccb29e695da48b17017e899a6119c7-2401x1000.png
  - Caption: Evaluator-optimizer workflow loop
  - License/attribution: Anthropic — "Building Effective Agents"

- `assets/external/building-effective-agents/autonomous-agent.png`
  - Source page: https://www.anthropic.com/research/building-effective-agents
  - Original URL: https://www-cdn.anthropic.com/images/4zrzovbb/website/58d9f10c985c4eb5d53798dea315f7bb5ab6249e-2401x1000.png
  - Caption: Autonomous agent flow (long-running agent loop)
  - License/attribution: Anthropic — "Building Effective Agents"

- `assets/external/azure-patterns/sequential-pattern.svg`
  - Source page: https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/ai-agent-design-patterns
  - Original URL: https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/_images/sequential-pattern.svg
  - Caption: Microsoft Azure — Sequential agent orchestration pattern
  - License/attribution: Microsoft Learn (Microsoft Docs CC-BY-4.0 content licence)

- `assets/external/azure-patterns/concurrent-pattern.svg`
  - Source page: https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/ai-agent-design-patterns
  - Original URL: https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/_images/concurrent-pattern.svg
  - Caption: Microsoft Azure — Concurrent agent orchestration pattern
  - License/attribution: Microsoft Learn (Microsoft Docs CC-BY-4.0 content licence)

- `assets/external/azure-patterns/group-chat-pattern.svg`
  - Source page: https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/ai-agent-design-patterns
  - Original URL: https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/_images/group-chat-pattern.svg
  - Caption: Microsoft Azure — Group-chat agent orchestration pattern
  - License/attribution: Microsoft Learn (Microsoft Docs CC-BY-4.0 content licence)

- `assets/external/azure-patterns/handoff-pattern.svg`
  - Source page: https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/ai-agent-design-patterns
  - Original URL: https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/_images/handoff-pattern.svg
  - Caption: Microsoft Azure — Handoff agent orchestration pattern
  - License/attribution: Microsoft Learn (Microsoft Docs CC-BY-4.0 content licence)

- `assets/external/azure-patterns/magentic-pattern.svg`
  - Source page: https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/ai-agent-design-patterns
  - Original URL: https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/_images/magentic-pattern.svg
  - Caption: Microsoft Azure — Magentic agent orchestration pattern
  - License/attribution: Microsoft Learn (Microsoft Docs CC-BY-4.0 content licence)

## R02 — Model Context Protocol

(no useful diagrams found — modelcontextprotocol.io renders its host/client/server diagram inline as Mermaid; the Anthropic announcement hero image exceeded the 2MB size cap)

## R03 — Anthropic Claude Code (headless agent runtime)

(no useful diagrams found — the official `code.claude.com` docs are text/code only; no architecture PNGs were exposed at the cited pages)

## R04 — Headless Coding Agents (Codex / Cursor / Gemini / OpenCode / Pi)

- `assets/external/codex/codex-cli-splash.png`
  - Source page: https://github.com/openai/codex
  - Original URL: https://github.com/openai/codex/raw/main/.github/codex-cli-splash.png
  - Caption: OpenAI Codex CLI splash / hero image
  - License/attribution: OpenAI (openai/codex repo, Apache-2.0)

- `assets/external/gemini-cli/gemini-cli-screenshot.png`
  - Source page: https://github.com/google-gemini/gemini-cli
  - Original URL: https://github.com/google-gemini/gemini-cli/raw/main/docs/assets/gemini-screenshot.png
  - Caption: Google Gemini CLI in action — terminal screenshot
  - License/attribution: Google (google-gemini/gemini-cli repo, Apache-2.0)

- `assets/external/opencode/screenshot.png`
  - Source page: https://github.com/sst/opencode
  - Original URL: https://raw.githubusercontent.com/sst/opencode/dev/packages/web/src/assets/lander/screenshot.png
  - Caption: OpenCode TUI — terminal UI screenshot
  - License/attribution: SST (sst/opencode repo, MIT)

- `assets/external/pi/tree-view.png`
  - Source page: https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/README.md
  - Original URL: https://pi.dev/tree-view.png
  - Caption: Pi coding agent — session tree-view screenshot
  - License/attribution: Mario Zechner / earendil-works (pi.dev), MIT

## R05 — OpenClaw + Agent Companies

(no useful diagrams found — openclaw/openclaw README only exposes logos and sponsor SVGs, and docs.openclaw.ai/gateway/heartbeat is text-only)

## R06 — Drizzle ORM + Embedded PostgreSQL

- `assets/external/drizzle/drizzle-studio.webp`
  - Source page: https://orm.drizzle.team/drizzle-studio/overview
  - Original URL: https://orm.drizzle.team/_astro/drizzle-studio.YeMH1cmC_15sm1a.webp
  - Caption: Drizzle Studio — visual DB management UI
  - License/attribution: Drizzle Team (orm.drizzle.team, Apache-2.0 docs)

- `assets/external/pglite/screenshot.png`
  - Source page: https://github.com/electric-sql/pglite
  - Original URL: https://raw.githubusercontent.com/electric-sql/pglite/main/screenshot.png
  - Caption: PGlite — embedded WASM PostgreSQL demo
  - License/attribution: ElectricSQL (electric-sql/pglite repo, Apache-2.0)

## R07 — pnpm workspace monorepo

(no useful diagrams found — pnpm.io renders its store/installation illustrations as inline base64 SVG inside the HTML, with no extractable absolute URL)

## R08 — React + Vite + shadcn/ui

- `assets/external/shadcn/dashboard-light.png`
  - Source page: https://ui.shadcn.com/  (block: dashboard-01)
  - Original URL: https://ui.shadcn.com/r/styles/new-york-v4/dashboard-01-light.png
  - Caption: shadcn/ui — official "dashboard-01" block, light theme
  - License/attribution: shadcn (ui.shadcn.com, MIT)

## R09 — SSE & Heartbeat patterns

- `assets/external/heartbeat/heartbeat-pattern.svg`
  - Source page: https://martinfowler.com/articles/patterns-of-distributed-systems/heartbeat.html
  - Original URL: https://martinfowler.com/articles/patterns-of-distributed-systems/index/Heartbeat.svg
  - Caption: Martin Fowler — HeartBeat pattern (Patterns of Distributed Systems)
  - License/attribution: Unmesh Joshi / Martin Fowler — martinfowler.com (per site terms; cite author)

## R10 — LLM cost accounting & budget enforcement

(no useful diagrams found — Anthropic / OpenAI / Gemini pricing pages are tables of text; LiteLLM, LangSmith, and FinOps Foundation pages did not expose attributable diagrams)

---

## 다운로드 요약

- Total images: 17
- Topics with images: 7  (R01, R04, R06, R08, R09, plus partial R02 fallback empty)
- Skipped due to size cap (>2MB): Anthropic MCP announcement hero image
- Skipped due to 403 / hotlink protection: Temporal blog ctfassets images (chain workflow / loop workflow)
- Skipped because rendered inline (no extractable URL): pnpm.io store/install diagrams,
  Vite dev-server diagrams, MCP architecture mermaid diagram

모든 외부 이미지는 원 출처 페이지·원본 URL과 함께 인용해야 하며, 보고서에서는
재배포가 아닌 *인용(quotation)* 으로 사용한다는 전제 하에 보관한다.
