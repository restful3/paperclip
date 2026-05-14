---
format: report
target: A4
topic: SSE & heartbeat patterns
---

# SSE 와 Heartbeat 패턴

## 1. Server-Sent Events 개요

**Server-Sent Events (SSE)** 는 단일 HTTP 연결 위에서 서버가 클라이언트로 이벤트를 푸시(push)하는 단방향 스트리밍 프로토콜이다. MDN 문서는 "This is a one-way connection, so you can't send events from a client to a server" 라고 명확히 못박는다([MDN](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events)). 클라이언트는 표준 **EventSource** 인터페이스 (`new EventSource(url)`) 로 연결을 열고, 서버는 `Content-Type: text/event-stream` 헤더와 함께 `event:`, `data:`, `id:`, `retry:` 필드를 가진 텍스트 프레임을 흘려보낸다([WHATWG HTML Living Standard](https://html.spec.whatwg.org/multipage/server-sent-events.html)).

핵심 운영 특성은 세 가지다. 첫째, **자동 재연결(automatic reconnection)** 이다. 연결이 끊기면 user agent 가 내장 재연결 절차를 수행하며, 재연결 시 `Last-Event-ID` 헤더로 마지막 수신 이벤트 ID 를 보내 서버가 그 지점부터 재개할 수 있게 한다. 스펙은 이전 시도가 실패한 경우 user agent 가 지수 백오프 지연을 도입할 수 있다고 허용하지만 표준 의무는 아니다([WHATWG](https://html.spec.whatwg.org/multipage/server-sent-events.html)). 둘째, **retry 필드**로 서버가 재연결 지연(ms)을 제어할 수 있다. 셋째, 콜론(`:`)으로 시작하는 라인은 주석으로 무시되며, 스펙은 "include a comment line every 15 seconds or so" 를 권장한다 — legacy proxy 가 idle 연결을 끊지 않도록 하는 **keepalive heartbeat** 용도다([WHATWG](https://html.spec.whatwg.org/multipage/server-sent-events.html)).

## 2. SSE vs WebSocket vs 폴링

| 항목 | SSE | WebSocket | 폴링 (polling) |
|---|---|---|---|
| 방향 | 서버 → 클라이언트 (단방향) | 양방향 | 클라이언트 → 서버 (요청-응답 반복) |
| 프로토콜 | 일반 HTTP/HTTPS | `ws://`/`wss://` (HTTP 업그레이드) | 일반 HTTP |
| 자동 재연결 | 표준 내장 | 직접 구현 필요 | N/A (매 요청 새 연결) |
| 메시지 포맷 | UTF-8 텍스트 (`text/event-stream`) | 텍스트/바이너리 자유 | 임의 |
| 프록시/방화벽 친화도 | 높음 (일반 HTTP) | 낮음 (업그레이드 차단되는 경우) | 가장 높음 |
| 지연 (latency) | 낮음 (push) | 가장 낮음 | 높음 (interval 만큼) |
| 서버 부하 | 낮음 | 낮음 | 높음 (유휴에도 요청) |

OpenAI, Anthropic 같은 LLM API 는 토큰 스트리밍에 모두 SSE 를 채택한다. Anthropic Messages API 문서는 `"stream": true` 일 때 "incrementally stream the response using server-sent events (SSE)" 라고 명시한다([Anthropic Messages Streaming](https://platform.claude.com/docs/en/api/messages-streaming)). 단방향 토큰 푸시에는 양방향 WebSocket 의 복잡도가 불필요하기 때문이다. HTTP/1.1 환경에서 브라우저당 동시 연결 6 개 제한이 있지만 HTTP/2 위에서는 기본 100 개 스트림으로 완화된다([MDN](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events)).

## 3. Heartbeat 와 워치독 패턴

**Heartbeat** 는 분산 시스템에서 노드/워커가 살아 있음을 주기적으로 신호(signal)로 알리는 패턴이다. Martin Fowler 의 *Patterns of Distributed Systems* 는 이를 "Periodically send a request to all the other servers indicating liveness of the sending server" 로 정의한다([martinfowler.com](https://martinfowler.com/articles/patterns-of-distributed-systems/heartbeat.html)). 폴링과의 차이는 **방향성** 이다 — heartbeat 는 push 기반(워커가 능동적으로 신호를 보냄), 폴링은 pull 기반(감독자가 매번 질의). 클러스터 규모가 커질수록 heartbeat 의 query overhead 가 더 적다.

장애 탐지는 **timeout 기반** 이다. 송신 간격을 `T` 라 하면 listener 는 `k·T` (보통 k=2\~3) 동안 신호가 없으면 장애로 판정한다. 간격은 네트워크 round-trip 시간을 초과해야 false positive 를 줄일 수 있다([martinfowler.com](https://martinfowler.com/articles/patterns-of-distributed-systems/heartbeat.html)).

**Liveness vs readiness** 구분도 중요하다. AWS Builders' Library 는 health check 를 "a way of asking a service on a particular server whether or not it is capable of performing work successfully" 로 정의하며, **liveness check** (TCP 연결, HTTP 200 같은 기본 도달성)와 **deep health check** (의존 서비스까지 검증)를 구분한다([AWS Builders' Library — Implementing health checks](https://aws.amazon.com/builders-library/implementing-health-checks/)). microservices.io 의 Health Check API 패턴은 `/health` 엔드포인트를 통해 "service instance might fail between health checks" 라는 한계를 인정하며 주기적 (periodic) 점검의 본질을 짚는다([microservices.io](https://microservices.io/patterns/observability/health-check-api.html)).

**Watchdog timer** 는 heartbeat 를 모니터링하는 감시자가 timeout 만료 시 워커를 재시작/우회하거나 작업을 다른 노드로 이관하도록 트리거하는 메커니즘이다. AWS 문서는 동시에 health check 가 fleet 전체에 false positive 를 일으켜 cascading outage 를 만들 위험도 경고한다([AWS](https://aws.amazon.com/builders-library/implementing-health-checks/)).

## 4. 장기 실행 에이전트의 체크포인트와 재개

Anthropic 의 *Building Effective Agents* 는 에이전트가 "potentially operate for many turns" 하면서 "Once the task is clear, agents plan and operate independently" 한다고 정리한다([Anthropic](https://www.anthropic.com/engineering/building-effective-agents)). 권장 패턴 중 **orchestrator-worker** 는 "a central LLM dynamically breaks down tasks, delegates them to worker LLMs, and synthesizes their results" 구조이며, 사전에 하위 작업을 결정할 수 없는 코딩·리서치 같은 도메인에 적합하다.

장기 실행 시 핵심은 **checkpoint** 다. 에이전트는 "pause for human feedback at checkpoints or when encountering blockers" 할 수 있어야 하며, 컨텍스트 윈도우 한계와 누적 오류(compounding errors)를 다루기 위해 sandboxed 환경 테스트와 guardrail 이 필수다([Anthropic](https://www.anthropic.com/engineering/building-effective-agents)). 즉 (1) 외부 저장소에 상태/투두 리스트를 직렬화해 컨텍스트 손실에 대비하고, (2) heartbeat/체크포인트 시점에 사람이나 감시 프로세스가 개입할 진입점을 둔다.

## 5. Paperclip 과의 관계

Paperclip 의 control plane 은 위 두 패턴을 그대로 합성한다. **Heartbeat scheduler** 가 정책에 따라 wakeup 을 enqueue 하고, 실행 경로가 run 을 claim 한 뒤 `adapter.execute(ctx)` 로 외부 런타임(Claude Code/Codex 등)을 한 turn 실행한 산출물을 task DB 에 기록한다 (`server/src/services/heartbeat.ts:7662`; 어댑터 호출 이벤트 타입명은 별개로 `adapter.invoke` 로 기록 — 같은 파일 `:7636`). 이것이 컨텍스트 윈도우를 넘어서는 long-running 실행을 가능케 하는 **체크포인트** 역할이다. 동시에 `server/src/realtime/live-events-ws.ts` 의 **WebSocket 채널** (`/api/companies/:companyId/events/ws`) 이 이 활동을 React 보드 UI 로 실시간 푸시해 사용자가 turn 단위 진행을 관찰할 수 있게 한다 — 즉 본 노트가 비교한 SSE 와 WebSocket 중 메인 보드는 양방향 keep-alive 가 자연스러운 WebSocket 을 택했고, SSE 는 plugin 브리지 스트림(`server/src/routes/plugins.ts`) 에 한정해 사용한다. 마지막으로 **silent-run watchdog** 은 `lastOutputAt` 기준 출력 silence 가 suspicion threshold 를 넘긴 active run 을 탐지해 stranded-work 회복 경로(`server/src/services/recovery/service.ts:676-760`, `doc/execution-semantics.md` 참조)로 보내며, 이는 Fowler 가 기술한 timeout 기반 장애 탐지를 출력 silence 시그널로 적용한 사례다.

## 6. 출처

- [MDN — Using server-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events)
- [WHATWG HTML Living Standard — Server-sent events](https://html.spec.whatwg.org/multipage/server-sent-events.html)
- [AWS Builders' Library — Implementing health checks](https://aws.amazon.com/builders-library/implementing-health-checks/)
- [microservices.io — Health Check API pattern](https://microservices.io/patterns/observability/health-check-api.html)
- [Martin Fowler — HeartBeat (Patterns of Distributed Systems)](https://martinfowler.com/articles/patterns-of-distributed-systems/heartbeat.html)
- [Anthropic — Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)
- [Anthropic Messages API — Streaming (SSE)](https://platform.claude.com/docs/en/api/messages-streaming)
