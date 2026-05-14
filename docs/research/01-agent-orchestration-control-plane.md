---
format: report
target: A4
topic: AI agent orchestration / control plane
---

# AI Agent Orchestration & Control Plane 패턴

## 1. 개요

생성형 AI 에이전트가 단일 챗봇에서 다중 에이전트(multi-agent) 시스템과 장기 실행(long-running) 워크플로로 확장되면서, 업계는 **에이전트 런타임(agent runtime)** 과 **제어 평면(control plane)** 을 별도로 다루기 시작했다. 런타임은 한 에이전트가 LLM 호출과 도구를 실행하는 루프를 책임지고, 제어 평면은 여러 에이전트·여러 벤더·여러 워크로드를 조직 차원에서 거버넌스·관측·예산 통제하는 상위 레이어다. Forrester는 2025년 12월 엔터프라이즈 에이전트 스택을 build / orchestrate / control 세 평면으로 분해하면서, 거버넌스가 "build와 orchestration 환경 바깥에" 위치해야 한다고 명시했다([Forrester, 2025](https://www.forrester.com/blogs/agent-control-planes-still-need-a-robust-standards-stack/)).

이 문서는 Paperclip이 속한 "control plane" 카테고리를 정의하고, LangGraph·Temporal·CrewAI·AutoGen·Claude Agent SDK 등 대표 구현이 채택한 **오케스트레이션 패턴(orchestration patterns)** — 오케스트레이터-워커, 계층형 매니저, 지속 실행(durable execution) — 을 정리한다. 또한 n8n·Airflow 같은 일반 워크플로 엔진과의 차이를 짚어, Paperclip이 왜 "회사형(company-shaped)" 제어 평면으로 분류되는지를 후속 보고서가 인용할 수 있는 근거를 제공한다.

## 2. 핵심 개념

| 용어 | 정의 | 출처 |
|---|---|---|
| **Agent runtime** | 단일 에이전트의 "context 수집 → 행동 → 검증 → 반복" 루프와 도구 실행을 처리하는 SDK·실행 환경. Claude Agent SDK는 이 루프를 추상화 뒤로 숨기지 않고 개발자에게 직접 노출한다. | [Claude Agent SDK overview](https://platform.claude.com/docs/en/agent-sdk/overview) |
| **Agent control plane** | 이종(heterogeneous) 에이전트들을 인벤토리·거버넌스·오케스트레이션하는 엔터프라이즈 레이어. Forrester 모델에서 build·orchestrate 평면 바깥에 위치한다. | [Forrester, 2025](https://www.forrester.com/blogs/agent-control-planes-still-need-a-robust-standards-stack/) |
| **Workflow vs Agent** | Anthropic은 workflows 를 사전 정의된 code path 로 orchestrate 되는 시스템으로, agents 를 LLM 이 자신의 process 와 tool 사용을 동적으로 지시하며 작업 완수 방식을 스스로 통제하는 시스템으로 구분한다. | [Anthropic, Building effective agents](https://www.anthropic.com/research/building-effective-agents) |
| **Orchestrator-worker** | 중앙 LLM이 동적으로 서브태스크를 분해해 워커 LLM에 위임하고 결과를 합성. 서브태스크가 사전 정의되지 않을 때 적합. | [Anthropic, Building effective agents](https://www.anthropic.com/research/building-effective-agents) |
| **Hierarchical process** | 매니저 에이전트가 능력 기반으로 태스크를 동적으로 할당·검토하는 구조. CrewAI의 `Process.hierarchical` 이 대표 사례. | [CrewAI Processes](https://docs.crewai.com/en/concepts/processes) |
| **Durable execution** | 워크플로가 핵심 지점마다 진행 상태를 저장해, 중단 후 정확히 그 지점부터 재개되는 기법. LangGraph는 checkpointer + thread_id로 구현. | [LangGraph durable execution](https://docs.langchain.com/oss/python/langgraph/durable-execution) |
| **Event History (Temporal)** | "When Temporal executes a Workflow, it records a full Event History — every single time code in the Workflow is run, every single time an Activity is called or returned, and more." 액티비티 재시도와 크래시 복구의 기반. | [Temporal blog](https://temporal.io/blog/durable-execution-meets-ai-why-temporal-is-the-perfect-foundation-for-ai) |

## 3. 주요 구현·레퍼런스 아키텍처

| 시스템 | 레이어 분류 | 핵심 패턴 | 비고 |
|---|---|---|---|
| **Anthropic Building Effective Agents** | 패턴 가이드(개념) | orchestrator-worker, prompt chaining, routing, evaluator-optimizer | "복잡한 프레임워크보다 단순·구성 가능한 패턴" 권장 ([Anthropic](https://www.anthropic.com/research/building-effective-agents)) |
| **Claude Agent SDK** | Agent runtime | 루프 노출형 SDK, in-process MCP custom tools, agentic search | Claude Code SDK가 Agent SDK로 리브랜딩, Python·TS 지원 ([overview](https://platform.claude.com/docs/en/agent-sdk/overview)) |
| **LangGraph (OSS) / LangGraph Platform** | Agent runtime + 일부 제어 | StateGraph, checkpointer 기반 durable execution, HITL interrupt | checkpointer + thread_id 로 workflow 진행 상태를 저장하고, 중단 시 마지막 기록된 step 부터 재개한다 ([docs](https://docs.langchain.com/oss/python/langgraph/durable-execution)) |
| **Temporal for AI** | Durable workflow 엔진 (런타임 하부 인프라) | Workflow + Activity, Event History, 자동 재시도, 장기 실행 | "Workflows automatically hold state over long periods of time (even years)" ([Temporal AI](https://temporal.io/solutions/ai)); OpenAI Codex 프로덕션 사용 사례 ([Temporal blog](https://temporal.io/blog/of-course-you-can-build-dynamic-ai-agents-with-temporal)) |
| **CrewAI / CrewAI AMP** | Multi-agent runtime + Enterprise 제어 | Sequential / Hierarchical process, manager_agent 동적 할당 | "Tasks are not pre-assigned; the manager allocates tasks to agents based on their capabilities" ([CrewAI docs](https://docs.crewai.com/en/concepts/processes)) |
| **Microsoft AutoGen / Agent Framework** | Multi-agent runtime | GroupChat 매니저가 다음 발화자를 선택, 계층형 채팅 | AutoGen은 maintenance mode, 후속은 Microsoft Agent Framework ([AutoGen GitHub](https://github.com/microsoft/autogen), [Agent Framework v1.0](https://devblogs.microsoft.com/agent-framework/microsoft-agent-framework-version-1-0/)) |
| **Azure AI Agent Design Patterns** | 패턴 가이드(개념) | sequential, concurrent, group chat, handoff, magentic | Microsoft Learn의 표준 분류 ([Azure Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/ai-agent-design-patterns)) |
| **n8n / Airflow** | Workflow automation (제어 평면 아님) | 사전 정의된 노드·DAG 실행, AI Agent 노드는 도구를 선택하며 완료까지 루프하는 reasoning step | LowCode Agency 비교 기사는 n8n 의 AI agent node 를 "an autonomous reasoning step that selects tools, loops until complete, and outputs a result" 로 설명하고, n8n workflow 에서 LangChain·CrewAI·AutoGen 같은 외부 agent framework 를 HTTP 로 호출할 수 있다고 명시한다 — 즉 control plane 은 아니지만 외부 agent runtime 과 연결 가능한 workflow automation 도구다 ([n8n 비교](https://www.lowcode.agency/blog/n8n-vs-agentic-ai)) |

레이어 관점에서 보면, Temporal은 LangGraph가 "agent runtime" 으로 동작할 수 있도록 그 아래에서 durable workflow를 제공할 수 있는 인프라이고, CrewAI·AutoGen은 multi-agent 협업 패턴을 코드 수준에서 표현하는 런타임이며, 제어 평면은 그 위에서 여러 런타임을 가로질러 거버넌스를 강제한다([Forrester, 2025](https://www.forrester.com/blogs/agent-control-planes-still-need-a-robust-standards-stack/)).

## 4. Paperclip과의 관계

Paperclip은 위 분류에서 **에이전트 런타임이 아니라 제어 평면(control plane only)** 에 해당한다. 깃허브 README는 Paperclip을 "a full control plane, not a wrapper" 로 명시하고, Claude Code·Codex·CLI 에이전트·HTTP 봇 등 이종 런타임을 어댑터로 연결한다([paperclipai/paperclip](https://github.com/paperclipai/paperclip)). 즉 LLM 호출 루프는 외부 런타임이 담당하고, Paperclip은 그 위에서 회사 메타포(roles, titles, reporting lines, permissions, budgets)에 따라 작업을 큐잉·분배·감사한다.

세부 패턴 측면에서 Paperclip의 차별점은 다음 세 가지다. 첫째, **단일 할당자(single-assignee) + 원자적 체크아웃(atomic checkout)**: "Atomic execution. Task checkout and budget enforcement are atomic, so no double-work and no runaway spend." ([README](https://github.com/paperclipai/paperclip)). 이는 GroupChat의 다중 발화자 모델이나 LangGraph의 다중 노드 동시 실행과 다르게, 한 작업은 동시 실행 락(execution lock)으로 한 에이전트만 점유한다. 둘째, **하트비트 스케줄링(heartbeat scheduling)**: DB 백드 wakeup 큐가 budget·workspace·secret·skill을 결합해 어댑터를 호출하므로, 장기 실행 에이전트가 이벤트 또는 크론 기반으로 깨어난다 — 이는 Temporal의 durable execution을 가볍게 모방한 형태다. 셋째, **하드 예산 정지(budget hard-stop)**: 비용은 company / agent / issue / project / goal / heartbeat run / provider / model 축으로 cost_events 테이블에 귀속되며(`packages/db/src/schema/cost_events.ts:9-30`), hard-stop 정책 평가·시작 차단·실행 취소는 company / agent / project 세 scope 에서 동작한다(`server/src/services/budgets.ts:647-862`, `server/src/services/heartbeat.ts:9431-9456`). 초과 시 해당 scope 가 pause 되고 활성 실행과 wakeup 큐가 함께 취소된다. n8n처럼 워크플로 단계만 실행하는 도구와 달리, Paperclip은 조직 트리(org tree)를 **권한과 예산이 흐르는 1급 객체**로 다룬다는 점에서 Forrester가 정의한 "vendor-agnostic governance layer" 정의와 정렬된다([Forrester](https://www.forrester.com/blogs/agent-control-planes-still-need-a-robust-standards-stack/)).

## 5. 출처

- [Anthropic — Building Effective Agents](https://www.anthropic.com/research/building-effective-agents)
- [Claude Agent SDK overview (Anthropic Docs)](https://platform.claude.com/docs/en/agent-sdk/overview)
- [Forrester — Agent Control Planes Still Need A Robust Standards Stack](https://www.forrester.com/blogs/agent-control-planes-still-need-a-robust-standards-stack/)
- [Microsoft Learn — AI Agent Orchestration Patterns (Azure Architecture Center)](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/ai-agent-design-patterns)
- [LangGraph — Durable execution docs](https://docs.langchain.com/oss/python/langgraph/durable-execution)
- [Temporal — Durable Execution meets AI](https://temporal.io/blog/durable-execution-meets-ai-why-temporal-is-the-perfect-foundation-for-ai)
- [Temporal for AI (solutions page)](https://temporal.io/solutions/ai)
- [Temporal — Of course you can build dynamic AI agents with Temporal](https://temporal.io/blog/of-course-you-can-build-dynamic-ai-agents-with-temporal)
- [CrewAI Docs — Processes (Sequential / Hierarchical)](https://docs.crewai.com/en/concepts/processes)
- [Microsoft AutoGen GitHub](https://github.com/microsoft/autogen)
- [Microsoft Agent Framework Version 1.0](https://devblogs.microsoft.com/agent-framework/microsoft-agent-framework-version-1-0/)
- [n8n vs Agentic AI — Lowcode Agency](https://www.lowcode.agency/blog/n8n-vs-agentic-ai)
- [Paperclip — paperclipai/paperclip GitHub](https://github.com/paperclipai/paperclip)
