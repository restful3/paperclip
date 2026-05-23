---
format: report
target: A4
topic: OpenClaw + Agent Companies spec
---

# OpenClaw 와 Agent Companies 명세

## 1. OpenClaw란

**OpenClaw**는 사용자가 자기 디바이스에서 직접 돌리는 오픈소스 **개인용 AI 어시스턴트(personal AI assistant)** 런타임으로, "Any OS. Any Platform. The lobster way." 라는 모토를 단다 ([openclaw/openclaw README](https://github.com/openclaw/openclaw)). 외침 "EXFOLIATE! EXFOLIATE!" 와 Node 24 권장 (최소 22.16+) 런타임 요건은 GitHub README 에 명시되어 있다 ([openclaw/openclaw](https://github.com/openclaw/openclaw)). 마스코트인 우주 바닷가재 **Molty** 와 **Peter Steinberger 및 community** 가 빌더로 명시된 곳은 공식 사이트 [openclaw.ai](https://openclaw.ai/) 이며, 문서는 [docs.openclaw.ai](https://docs.openclaw.ai/gateway/heartbeat) 에 있다. 실행 환경은 Node 24 권장(최소 22.16+) 이며, 에이전트가 채널(Slack, Gmail 등)을 거쳐 사용자에게 응답하는 형태다.

**워크스페이스 파일 시스템(workspace files)** 은 OpenClaw 정체성의 핵심이다 — 에이전트는 평범한 마크다운 파일 묶음으로 정의된다 ([Capodieci, "OpenClaw Workspace Files Explained"](https://capodieci.medium.com/ai-agents-003-openclaw-workspace-files-explained-soul-md-agents-md-heartbeat-md-and-more-5bdfbee4827a)).

- **SOUL.md** — 에이전트의 인격(personality), 가치관, 톤, 행동 경계를 정의하는 필수 파일. 모든 세션 시작 시 시스템 프롬프트에 주입된다 ([aaronjmars/soul.md](https://github.com/aaronjmars/soul.md)).
- **HEARTBEAT.md** — 선택적 "깨어남 체크리스트" 다. 기본 30 분 (Anthropic OAuth 사용 시 1 시간) 간격으로 에이전트가 자동으로 깨어나 이 파일을 읽고, 점검할 게 없으면 `HEARTBEAT_OK` 만 반환한다. `isolatedSession: true` 옵션으로 매 heartbeat 마다 새 세션을 띄워 토큰 비용을 \~100K → 2\~5K 로 줄일 수도 있다 ([OpenClaw Heartbeat 문서](https://docs.openclaw.ai/gateway/heartbeat)).
- **AGENTS.md / TOOLS.md** — 에이전트 설정과 사용 가능 툴을 기술한다 ([openclaw/openclaw README](https://github.com/openclaw/openclaw)).

OpenClaw 자체에는 **Gateway** 가 내장되어 있어 세션·채널·툴·이벤트의 control plane 역할을 하고, webhook 자동화·cron·Gmail Pub/Sub 같은 트리거를 지원한다 ([openclaw/openclaw README](https://github.com/openclaw/openclaw)).

## 2. Agent Companies (agentcompanies/v1) 명세

**Agent Companies 명세(agentcompanies/v1)**는 회사·팀·에이전트·프로젝트·태스크·스킬을 **YAML frontmatter 가 붙은 마크다운 파일 묶음**으로 표현하는 "filesystem- and GitHub-native" 패키지 포맷이다 ([companies-spec.md](https://github.com/paperclipai/paperclip/blob/master/docs/companies/companies-spec.md)). 정식 schema 식별자는 현재 `agentcompanies/v1-draft` 다.

**여섯 가지 패키지 종류(kinds)** 가 있고, 각각이 루트 마크다운 파일로 식별된다 ([companies-spec.md](https://github.com/paperclipai/paperclip/blob/master/docs/companies/companies-spec.md)):

- `COMPANY.md` — 회사 단위 패키지
- `TEAM.md` — 조직 서브트리
- `AGENTS.md` — 개별 에이전트 정의
- `PROJECT.md` — 경량 프로젝트 묶음
- `TASK.md` — 시작 태스크 또는 반복 작업
- `SKILL.md` — 스킬 패키지 (Agent Skills 명세 준수)

**관용적 폴더 레이아웃(conventional folder layout)** 은 `agents/<slug>/AGENTS.md`, `teams/<slug>/TEAM.md`, `projects/<slug>/PROJECT.md` 와 그 아래 `tasks/<slug>/TASK.md`, `skills/<slug>/SKILL.md` 를 두고, 보조로 `assets/`, `scripts/`, `references/` 디렉터리를 둔다 ([companies-spec.md](https://github.com/paperclipai/paperclip/blob/master/docs/companies/companies-spec.md)). 벤더 확장(vendor extension) 은 `.paperclip.yaml` 사이드카에 격리된다.

**공통 frontmatter 필드** 는 `schema`, `kind`, `slug`, `name`, `description`, `version`, `license`, `authors`, `homepage`, `tags`, `metadata`, `sources` 다. 핵심 원칙은 "마크다운이 표준(canonical)", "git 저장소 자체가 유효한 패키지 컨테이너", "레지스트리는 선택적 디스커버리 레이어" 라는 것이다 ([companies-spec.md](https://github.com/paperclipai/paperclip/blob/master/docs/companies/companies-spec.md)). 즉, npm/PyPI 같은 중앙 레지스트리에 의존하지 않고 GitHub URL 만으로 회사를 배포·import 할 수 있다.

별도 카탈로그 [paperclipai/companies](https://github.com/paperclipai/companies) 는 16 개의 사전 빌드 회사, 440+ 에이전트, 500+ 스킬을 모은 **레퍼런스 구현 모음** 이며, 현재 CLI 구현 기준 `npx paperclipai company import <path-or-github-url>` 처럼 positional 소스 인자를 받는다 (`cli/src/commands/client/company.ts:1268-1271`). 카탈로그 README 가 보여 주는 `--from` 옵션 예시는 CLI 의 현재 시그니처와 다르므로, README 예시를 인용할 때는 그 차이를 분리해서 표기한다 ([paperclipai/companies README](https://github.com/paperclipai/companies)).

## 3. Paperclip ↔ OpenClaw ↔ Agent Companies 관계

세 개체의 위상은 다음과 같이 정리된다.

- **Paperclip** = "human control plane for AI labor", 즉 회사·태스크·이슈 오케스트레이션 control plane ([paperclip.ing](https://paperclip.ing/), [paperclipai/paperclip](https://github.com/paperclipai/paperclip)).
- **OpenClaw** = Paperclip 이 위탁할 수 있는 **실행 런타임 한 종류** (어댑터 디렉터리에 `acpx-local`, `claude-local`, `codex-local`, `cursor-cloud`, `cursor-local`, `gemini-local`, `grok-local`, `opencode-local`, `openclaw-gateway`, `pi-local` 10 개 패키지가 들어 있고 registry built-in 으로 등록된다 — `server/src/adapters/registry.ts:511-524`). OpenClaw 자체는 Paperclip 없이도 독립 동작한다 ([openclaw/openclaw](https://github.com/openclaw/openclaw)).
- **Agent Companies (agentcompanies/v1)** = Paperclip 이 정의·홍보하는 **회사 패키지 포맷(portability format)**. Paperclip 저장소(`docs/companies/companies-spec.md`) 에 명세가 산다는 점에서 사실상 **Paperclip 주도 명세** 지만, 마크다운+frontmatter 라는 중립적 형식이라 다른 오케스트레이터가 import 하는 것을 막지 않는다 ([companies-spec.md](https://github.com/paperclipai/paperclip/blob/master/docs/companies/companies-spec.md)).

요약하면, Paperclip 의 슬로건 "OpenClaw 가 직원이라면 Paperclip 은 회사" 는 다음 매핑을 의도한다 — OpenClaw 는 *한 명의 AI 직원을 실행* 하는 런타임이고, Paperclip 은 *그런 직원들을 회사 단위로 모아 일을 분배* 하는 상위 레이어이며, Agent Companies 명세는 그 회사의 *조직도와 직무 기술서를 git 으로 옮길 수 있게 만드는 파일 포맷* 이다. 세 프로젝트 모두 **paperclipai / openclaw GitHub org** 와 **Peter Steinberger 커뮤니티** 라는 공통 인적 궤도를 공유하는 것으로 보이나, 자매 프로젝트(sister projects) 라는 공식 표현 자체는 공개 자료 미확인.

## 4. Paperclip 어댑터로 OpenClaw 연결

Paperclip 의 `packages/adapters/openclaw-gateway` (npm 명: `@paperclipai/adapter-openclaw-gateway`) 는 OpenClaw 의 Gateway 프로토콜을 호출해 두 시스템을 잇는다. 전송은 HTTP 가 아니라 **WebSocket(ws:// 또는 wss://)** 전용이며, 핸드셰이크는 `connect.challenge` 수신 → `req connect` (프로토콜·클라이언트·auth·device payload) → `req agent` → `req agent.wait` 로 완료를 기다린 뒤 이벤트 프레임을 스트리밍하는 순으로 진행된다 ([openclaw-gateway adapter](https://github.com/paperclipai/paperclip/tree/master/packages/adapters/openclaw-gateway)). Paperclip 의 `runId` 가 idempotency key 로, 세션 키는 issue/fixed/run 라우팅 전략으로 결정되며, 디바이스는 Ed25519 키페어 서명으로 인증된다. 에이전트의 출력은 `[openclaw-gateway:event] run=<id> stream=<stream> data=<json>` 형태의 구조화 로그로 Paperclip 측 UI/CLI 에 transcript 로 렌더된다 — 이것이 사실상의 "agent → Paperclip pingback" 채널이다.

## 5. 출처

- [openclaw/openclaw (GitHub)](https://github.com/openclaw/openclaw)
- [OpenClaw Heartbeat 공식 문서](https://docs.openclaw.ai/gateway/heartbeat)
- [aaronjmars/soul.md (SOUL.md 빌더)](https://github.com/aaronjmars/soul.md)
- [mergisi/awesome-openclaw-agents (162 SOUL.md 템플릿)](https://github.com/mergisi/awesome-openclaw-agents)
- [Roberto Capodieci, "OpenClaw Workspace Files Explained"](https://capodieci.medium.com/ai-agents-003-openclaw-workspace-files-explained-soul-md-agents-md-heartbeat-md-and-more-5bdfbee4827a)
- [paperclipai/paperclip (메인 저장소)](https://github.com/paperclipai/paperclip)
- [Agent Companies 명세 (companies-spec.md)](https://github.com/paperclipai/paperclip/blob/master/docs/companies/companies-spec.md)
- [AGENTCOMPANIES_SPEC_INVENTORY.md](https://github.com/paperclipai/paperclip/blob/master/doc/AGENTCOMPANIES_SPEC_INVENTORY.md)
- [OPENCLAW_ONBOARDING.md](https://github.com/paperclipai/paperclip/blob/master/doc/OPENCLAW_ONBOARDING.md)
- [paperclipai/companies (사전 빌드 회사 카탈로그)](https://github.com/paperclipai/companies)
- [openclaw-gateway adapter](https://github.com/paperclipai/paperclip/tree/master/packages/adapters/openclaw-gateway)
- [Paperclip 공식 사이트](https://paperclip.ing/)

**불확실 / 공개 자료 미확인 항목**

- Paperclip · OpenClaw · agentcompanies 가 "자매 프로젝트(sister projects)" 라거나 공식적으로 동일 메인테이너 그룹이라는 명시적 선언 — 공개 자료 미확인 (조직 GitHub 명·메인테이너 이름이 겹친다는 정황 증거만 있음).
- `agentcompanies/v1` 가 `v1-draft` 에서 정식 `v1` 으로 승격되었는지 시점 — 명세 본문은 "draft" 표기를 유지 중 ([companies-spec.md](https://github.com/paperclipai/paperclip/blob/master/docs/companies/companies-spec.md)).
- OpenClaw `OPENCLAW_ONBOARDING.md` 본문에 SOUL.md / HEARTBEAT.md 직접 언급 — 해당 onboarding 문서에서는 발견되지 않았고, 두 파일의 의미는 OpenClaw 자체 문서·커뮤니티 자료에서 보강함.
