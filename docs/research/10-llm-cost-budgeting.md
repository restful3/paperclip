---
format: report
target: A4
topic: LLM cost accounting & budget enforcement
---

# LLM 토큰 비용 회계와 예산 강제

## 1. 모델 가격 구조

> **검증일 / 가격 변동성 노트.** 아래 단가는 **2026-05-09 기준** 각 공급자 공식 페이지의 *예시 모델* 표본이다. LLM 가격표는 분기 단위로 갱신되므로, 각 모델명을 그대로 인용해도 실제 단가는 달라질 수 있다. 회계 시스템은 *단가 표본* 이 아니라 *공급자 응답 `usage`* 를 권위 있는 입력으로 삼아야 한다.

LLM 가격은 거의 모든 주요 공급자가 **백만 토큰 단가(MTok)** 기준으로 input·output을 분리하고, 여기에 **prompt cache** 할인 단가를 추가하는 3축 모델로 수렴했다. Anthropic의 경우 **Claude Sonnet 4.6**은 base input $3 / 5분 cache write $3.75 (1.25x) / 1시간 cache write $6 (2x) / **cache hit $0.30 (0.1x)** / output $15 (per MTok)로 구성된다 ([Anthropic Pricing](https://platform.claude.com/docs/en/about-claude/pricing), 2026-05-09 표본). cache read가 base의 10%에 불과하므로 5분 cache는 한 번만 재사용해도 손익분기를 넘는다 ([Anthropic Prompt Caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)).

OpenAI 의 [API Pricing](https://openai.com/api/pricing/) 페이지(2026-05-09 확인)는 GPT-5.x 라인업을 전면에 두고, **cached input 은 base 의 약 10%** 로 할인된다. 즉 캐시 할인 폭이 Anthropic 의 cache hit($0.30 / 0.1×) 과 같은 수준으로 수렴해 있다(과거 GPT-4o 시점의 base 50% 수준 캐시 할인은 *역사 예시* 로 분리한다 — input $2.50 / cached input $1.25 / output $10 per MTok 가 그 표본이었다). Google 의 **Gemini 2.5 Pro** 는 입력 길이로 가격을 분기시키는 독특한 구조다. 200k 토큰 이하는 input $1.25 / cached $0.125 / output $10 이고, 200k 초과 프롬프트는 input $2.50 / cached $0.25 / output $15 로 두 배 단가가 적용되며, **context caching 은 token 단가 외에 storage 단가 $4.50 / 1M tokens / hour 가 별도로 청구된다** ([Gemini API Pricing](https://ai.google.dev/gemini-api/docs/pricing), 2026-05-09 표본). 결과적으로 **회계 시스템은 모델·공급자·캐시 종류·캐시 storage 시간** 별로 단가를 따로 보관해야 하며, 단일 "$/token" 칼럼만으로는 청구가 정확히 재현되지 않는다.

또한 Anthropic의 batch API와 Gemini의 batch tier는 input/output에 일률적으로 50% 할인을 적용해 기존 가격 위에 곱해지는 modifier로 동작한다 ([Anthropic Pricing - Batch](https://platform.claude.com/docs/en/about-claude/pricing)).

## 2. 토큰 카운팅

토큰 수 산정은 모델별 tokenizer가 다르므로 **공급자가 반환한 usage 필드를 권위 있는(authoritative) 출처로 사용**해야 한다. OpenAI는 `tiktoken` 라이브러리를 공개하고 있으며, GPT-4o 계열은 `o200k_base`, GPT-3.5/4는 `cl100k_base` 인코딩을 쓴다 ([tiktoken GitHub](https://github.com/openai/tiktoken)). Anthropic은 사전 계산용 무과금 엔드포인트 `POST /v1/messages/count_tokens` 를 제공해 system prompt·tools·images·cache breakpoint 까지 포함된 토큰 수를 호출 전에 산출할 수 있어 **예산 사전 검증(pre-flight check)** 에 적합하다 ([Anthropic Count Tokens](https://platform.claude.com/docs/en/api/messages-count-tokens)). 한편 Claude Opus 4.7은 새 tokenizer를 도입해 동일 텍스트 기준 최대 35% 더 많은 토큰을 사용할 수 있어 ([Anthropic Pricing](https://platform.claude.com/docs/en/about-claude/pricing)), 어림 추정(approximation)에 의존하면 5\~15% 의 누적 오차가 발생한다 ([Zop.dev LLM FinOps](https://zop.dev/resources/blogs/llm-finops-per-feature-token-budget/)). 회계 신뢰성을 위해서는 응답 객체의 `usage.input_tokens`, `usage.output_tokens`, `usage.cache_read_input_tokens`, `usage.cache_creation_input_tokens` 를 그대로 저장하는 것이 표준이다.

## 3. 비용 관측 도구

3rd-party 관측 도구들은 공통적으로 **(요청 단위 raw usage) + (귀속 메타데이터) + (단가 테이블 기반 비용 계산)** 의 3계층을 갖는다. **Helicone**은 `Helicone-User-Id`, `Helicone-Session-Id`, 임의의 `Helicone-Property-*` 헤더를 통해 user·session·feature·workflow 단위로 비용을 분해할 수 있게 한다 ([Helicone Custom Properties](https://docs.helicone.ai/features/advanced-usage/custom-properties)). **LiteLLM**은 게이트웨이 자체가 100+ LLM의 단가를 내장해 spend log에 hashed API key, internal user, team_id, model, prompt/completion/total tokens, USD spend, custom metadata를 적재하며, key/team/model 단위 `max_budget`과 `soft_budget_cooldown` 을 지원한다 ([LiteLLM Cost Tracking](https://docs.litellm.ai/docs/proxy/cost_tracking)). **LangSmith** 는 trace tree 기반으로 inputs / outputs / metadata 를 기록하고 ([LangSmith Observability Concepts](https://docs.langchain.com/langsmith/observability-concepts)), 비용은 token counts · model/provider · model price 를 입력으로 한 model pricing map 으로 자동 산정한다 ([LangSmith Cost tracking](https://docs.langchain.com/langsmith/cost-tracking)). 즉, **저장은 토큰 단위·과금은 단가 테이블 lookup** 이라는 분리 원칙이 사실상의 표준이다.

## 4. 예산 강제 패턴

FinOps 분야는 LLM 비용에 대해 **4단 레버 — tag every call, count tokens authoritatively, aggregate per feature, enforce per-feature budget** — 을 정착시키고 있다 ([Zop.dev LLM FinOps](https://zop.dev/resources/blogs/llm-finops-per-feature-token-budget/)). 강제(enforcement) 계층은 일반적으로 graduated stage로 구성된다.

- **Soft alert**: 월 예산의 50%·80%·100% 임계치에서 알림 발송. 사용자 차단 없이 가시성만 부여.
- **Hard cap**: 100% 초과 시 게이트웨이가 후속 요청을 429로 거절하는 hard ceiling.
- **Session ceiling**: 단일 agent 세션의 누적 토큰이 임계치를 넘으면 다음 LLM 호출 전에 세션을 종료.
- **Billing code / chargeback**: 모든 요청에 `feature_id` 또는 OpenAI `user` 파라미터 같은 태그를 부여해 사후에 팀·기능·테넌트 단위로 chargeback 회계를 돌린다 ([FinOps Invoicing & Chargeback](https://www.finops.org/framework/capabilities/invoicing-chargeback/)).
- **Safety valve**: 110% burst 일일 허용·인프라 장애 시 soft-fail·인시던트용 명시적 override 절차.

## 5. Paperclip과의 관계

Paperclip의 `cost_events` 스키마(`/home/restful3/workspace/paperclip/packages/db/src/schema/cost_events.ts`)는 정확히 본 보고서가 정리한 표준 — 즉 **input_tokens / cached_input_tokens / output_tokens** 를 분리 저장하고 `cost_cents`를 별도 칼럼으로 갖는 raw usage 모델 — 을 따르며, `provider`·`model`·`biller`·`billing_type` 으로 공급자 단가·청구 주체·과금 종류를 함께 기록한다. `billing_code` 는 `cost_events`·`issues` 양쪽에 같이 달리는 자유 형식 라벨로, 외부 회계 시스템과의 매핑 키 또는 이슈 트리 묶음 식별자로 쓰이며 *상위 요청자에게 자동 롤업되는 키는 아니다*. 프로젝트 단위 attribution 은 `costs.byProject` (`server/src/services/costs.ts:454-502`) 가 `coalesce(cost_events.project_id, run→activity_log→issue.project_id)` 순으로 결정한다. `company_id`/`agent_id`/`issue_id`/`project_id`/`goal_id`/`heartbeat_run_id` 외래키를 통해 **회사 → 프로젝트 → 이슈 → agent → heartbeat** 의 다축 롤업이 가능하며, `budget_policies`는 soft alert·hard cap 임계치를 정의하고, `budget_incidents`는 hard cap 발동 시 자동 일시정지(auto-pause)된 사건을 기록해 §4의 graduated enforcement 패턴을 그대로 구현한다.

## 6. 출처

- [Anthropic Pricing](https://platform.claude.com/docs/en/about-claude/pricing)
- [Anthropic Prompt Caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
- [Anthropic Count Tokens API](https://platform.claude.com/docs/en/api/messages-count-tokens)
- [OpenAI API Pricing 2026](https://openai.com/api/pricing/)
- [Gemini API Pricing](https://ai.google.dev/gemini-api/docs/pricing)
- [tiktoken GitHub](https://github.com/openai/tiktoken)
- [Helicone Custom Properties](https://docs.helicone.ai/features/advanced-usage/custom-properties)
- [LiteLLM Cost Tracking](https://docs.litellm.ai/docs/proxy/cost_tracking)
- [LangSmith Observability Concepts](https://docs.langchain.com/langsmith/observability-concepts)
- [LangSmith Cost tracking](https://docs.langchain.com/langsmith/cost-tracking)
- [Zop.dev — LLM FinOps: Per-Feature Token Budgets](https://zop.dev/resources/blogs/llm-finops-per-feature-token-budget/)
- [FinOps Foundation — Invoicing & Chargeback](https://www.finops.org/framework/capabilities/invoicing-chargeback/)
