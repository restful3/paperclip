---
format: report
target: A4
topic: 강점 · 약점 · 트레이드오프 (09)
---

# Pros & Cons — Paperclip의 트레이드오프

## 1. 평가 프레임

이 장은 한 가지 질문에 답한다 — *언제 Paperclip을 쓰고, 언제 쓰지 않아야 하는가?* 평가는 다음 6개 축에서 이뤄진다.

1. **개념적 명료성** — 무엇을 푸는지가 명확한가
2. **데이터 모델의 견고성** — 작업·비용·org가 일관되게 모델링됐는가
3. **운영 안전망** — 자동화 대비 인간 통제의 균형
4. **확장성** — 새 어댑터·플러그인을 코어 수정 없이 추가 가능한가
5. **운영 부담** — 설치·업데이트·모니터링의 무게
6. **성숙도** — 누락된 영역과 향후 리스크

## 2. 강점

**표 1. Paperclip의 강점**

| 강점 | 코드/문서 근거 | 왜 중요한가 |
|---|---|---|
| **명확한 책임 분리 — control plane vs execution plane** | 장기 SPEC `invoke/status/cancel`(`doc/SPEC.md:207-215`) vs 현재 구현 `execute`+`testEnvironment` 필수(`packages/adapter-utils/src/types.ts:349-352`) — 어댑터 계약이 *두 층*으로 얇게 유지된다 | 새 런타임이 등장해도(예: GPT-5 native CLI) 어댑터만 추가하면 된다. 코어가 *에이전트 자체를 흉내내지 않는다*는 점이 장기 유지 부담을 줄인다. |
| **회사를 1급 객체로** | 80여 개 테이블 중 60여 개가 `company_id` 컬럼 보유, multi-company 격리 | 한 인스턴스로 여러 자율 사업체를 운영하는 시나리오가 자연스럽다. 데이터 격리가 강제되어 보안 회귀가 적다. |
| **단일 assignee + atomic checkout** | `issues.checkout_run_id`를 중심으로 한 조건부 UPDATE (`server/src/services/issues.ts:4659-4809` — 의사코드는 02장 §5) | 동시성 문제를 데이터 모델 차원에서 해결. CRDT/낙관적 락 없이 다중 조건 UPDATE 한 번으로 lock + status + adopt. |
| **보수적 회복 모델** | `doc/execution-semantics.md:407-422` §12 + `doc/SPEC-implementation.md:434-442` V1 liveness rule — *"preserve ownership / retry once / explicit recovery action / escalate visibly"* | 자동 재할당의 위험(잘못된 할당, 중복 작업)을 거부하고, 명시적인 회복 이슈로 보드/회복 오너에게 결정권을 돌린다. |
| **Token-level 비용 회계** | `cost_events.cached_input_tokens` 별도 컬럼 + `project_id`/`heartbeat_run_id` 다축 FK · `billing_code` 자유 라벨 | 프롬프트 캐싱이 도입된 2024+ LLM 가격 모델을 정확히 반영. byProject attribution은 `coalesce(cost_events.project_id, run→activity_log→issue.project_id)` 경로로 계산되며, 회사 단위 burn rate가 신뢰할 만하다. |
| **하드 일시정지 안전장치** | hard ceiling 정책(`doc/SPEC.md:295-300`) + `budgetService.getInvocationBlock`(`server/src/services/heartbeat.ts:8610-8618`) · `withAgentStartLock`(`server/src/services/agent-start-lock.ts:32`) · `cancelBudgetScopeWork`(`server/src/services/heartbeat.ts:9431-9456`) 세 메커니즘 조합 | 폭주 루프에 의한 토큰 사고를 차단하는 안전망. *V1의 must-have*로 우선순위가 높다. |
| **단일 origin dev 흐름** | Vite middlewareMode + `pnpm dev` 한 줄 | 기여 진입 장벽이 낮다. CORS/세션 잡음 없이 즉시 시작. |
| **풀 감사 로그 + WebSocket 라이브 피드** | `activity_log` 불변 + `/api/companies/:companyId/events/ws` | 비동기 분산 에이전트 환경에서 *지금 무슨 일이 일어나는가*를 사람이 따라잡을 수 있다. |
| **포터블 회사 (template/snapshot)** | `services/company-portability.ts` (178 KB) | 회사 설정 자체가 export/import 가능 — Agent Companies spec/v1의 기반. |
| **플러그인 SDK + 어댑터 외부화** | packages/plugins, \~/.paperclip/adapter-plugins.json | "knowledge base · 매출 회계 · 새 어댑터" 같은 V1 범위 밖 기능을 코어 수정 없이 도입할 수 있다. |
| **무설정 dev DB** | embedded-postgres 18, 자동 migrations | 첫 사용자의 시간 1\~2일을 절약. PGlite 보다 호환성 우선. |

표 1의 11개 강점은 *세 군집*으로 묶인다 — (a) **모델 명료성**(1\~3행: 책임 분리, 회사 1급화, atomic checkout)이 가장 위쪽에 위치한 것은 우연이 아니다. 이 셋이 무너지면 나머지 강점이 무너지기 때문이다. (b) **운영 안전망**(4\~6행: 보수적 회복, 토큰 회계, 하드 일시정지)은 자율 에이전트 회사가 폭주하지 않도록 막는 *결정의 묶음*이다. (c) **개발자/운영자 경험**(7\~11행: 단일 origin, 감사 로그+라이브, 포터블, 플러그인, 무설정 dev)은 *없어도 시스템은 돌지만 사용자가 떠나는* 종류의 강점이다. Paperclip의 강점은 *"모델은 명료하고, 안전망은 단단하고, 진입은 쉽다"*로 압축된다.

## 3. 약점 / 미성숙 영역

**표 2. 약점과 그 원인**

| 약점 | 어디서 드러나는가 | 영향 |
|---|---|---|
| **거대 단일 파일들** | `routes/issues.ts` 205 KB, `services/issues.ts` 201 KB, `services/company-portability.ts` 178 KB, `ui/pages/IssueDetail.tsx` 163 KB, `routes/access.ts` 140 KB, `routes/agents.ts` 119 KB, `routes/plugins.ts` 93 KB | 가독성·테스트성 저해. 신규 기여자가 한 번에 구조를 파악하기 어렵다. 서비스 분할이 점진적 과제. |
| **자동 재할당 부재** | execution-semantics §12의 *"does not automatically reassign"* | 에이전트 한 명이 사라지면 보드가 직접 처리해야 한다. 본질적으로 *의도된* 약점이지만 실수로 사람이 떠난 경우 시스템이 그걸 *알려 줄* 뿐 *해결*하지 않는다. |
| **고급 거버넌스 부재 (V1)** | 회사 멤버십(`packages/db/src/schema/company_memberships.ts`)과 4 단계 휴먼 role(owner/admin/operator/viewer, `packages/shared/src/constants.ts:624-630`)은 이미 있으나, `approvalsNeeded > 1` 다중 승인이 거부되고(`server/src/__tests__/issue-execution-policy.test.ts:98-105`) multi-member boards · advanced governance 는 Not V1(`doc/SPEC.md:489-493`) | 4-eye 결재·승인 위임·다중 멤버 보드를 *코어*에서 표현하기 어렵다. 엔터프라이즈 거버넌스가 즉시 필요한 환경은 플러그인 또는 외부 정책 엔진으로 보강해야 한다. |
| **지식 베이스 부재** | SPEC §11 anti-goal | 에이전트 간 *공유된 사실/문서*가 task system + comments 외에 없다. RAG 같은 KB가 필요한 도메인은 플러그인으로 자체 해결해야 한다. |
| **외부 매출/비용 회계 부재** | SPEC §12 anti-requirement | "회사가 돈을 벌고 있나?" 라는 질문을 시스템이 안다고 보장하지 않는다. 토큰 비용만 추적. |
| **자동 모델 가격 업데이트 X** | `cost_events.cost_cents` 는 인입 시점에 산정 | 가격 모델이 자주 바뀌는 LLM 시장에서, 가격표 갱신을 누가/어떻게 책임지는지 명확치 않다. |
| **OAuth/SSO 부재** | Better Auth 세션 + 이메일 기반 | 엔터프라이즈 도입 시 필수인 IdP 통합이 V1 에 없다. |
| **모바일 UI 미완성** | "Mobile Ready" 슬로건 vs `Inbox.tsx` 101 KB의 데스크톱 위주 디자인 | README가 약속한 모바일 모니터링이 일부 페이지에 한정. |
| **Cost dashboard의 시각화 한계** | `Costs.tsx` 48 KB 단일 페이지 | 분석 깊이(코호트, 추세, 모델 mix)가 LangSmith 등 전문 도구 대비 얕다. |
| **공식 클라우드 마켓플레이스 미정** | "ClipHub coming soon" | 사용자 간 회사 템플릿 교환이 느슨하다. |

표 2의 약점들은 *시간 차원*에서 두 종류로 나뉜다 — *V1 의도적 범위 밖 항목* (자동 재할당 부재, 고급 거버넌스(4-eye/다중 승인) 부재, KB 부재, 매출 회계 부재, OAuth/SSO, 클라우드 마켓플레이스)과 *시간이 흐르면 자연스럽게 줄어들 미성숙* (거대 단일 파일, 자동 가격 업데이트, 모바일 UI, Cost dashboard 깊이)이다. 첫 묶음을 만나면 기다리거나 직접 플러그인으로 메우는 선택지가 있고, 두 번째 묶음은 프로젝트 성숙도와 함께 해소될 가능성이 크다.

심각도 기준으로는 **거대 단일 파일**과 **자동 가격 업데이트 부재**가 가장 즉각적인 운영 리스크다. 전자는 새 기여자가 이슈 lifecycle을 수정할 때 회귀를 만들 가능성을 높이고, 후자는 비용 대시보드의 신뢰도를 직접 흔든다. 반면 **KB 부재**와 **매출 회계 부재**는 Paperclip의 정체성 자체를 깨는 결함이라기보다 의도적으로 얇은 코어를 유지한 결과다. 따라서 보완 우선순위는 "코어 신뢰도를 해치는 미성숙"을 먼저 줄이고, "도메인별 확장"은 플러그인 또는 외부 시스템 연동으로 푸는 쪽이 맞다.

## 4. 트레이드오프 — 의도된 절충

다음은 약점이 아니라 *결정* 이다.

**표 3. 의도된 트레이드오프**

| 결정 | 대안 | Paperclip의 선택 이유 |
|---|---|---|
| 단일 assignee · 다중 조건 atomic checkout | 다수 assignee · 락 free | 단순성. 동시성 분쟁이 데이터 모델 + 한 번의 조건부 UPDATE 로 사라진다. |
| 자동 회복은 한 번만 | 무한 재시도 또는 다른 에이전트로 재할당 | 잘못된 자동화의 위험(잘못된 오너, 중복 비용) 회피. 모호하면 사람에게 드러낸다. |
| Tasks + comments 만, 별도 채팅 X | 별도 메신저(Slack-like) | 모든 컨텍스트가 작업에 부착되어 감사 일치성 확보. 인박스 파편화 회피. |
| process / http 같은 범용 어댑터 보존 | 풍부한 어댑터만 | "호출 가능한 것은 모두 채용된다"는 약속을 깨지 않기 위해. |
| 멀티-회사 단일 인스턴스 | 회사당 인스턴스 | 운영 부담 절감. 인스턴스를 하나만 돌보면 된다. |
| Drizzle 스키마-퍼스트 | Prisma 같은 schema DSL | TypeScript-native, SQL-like, 마이그레이션 미세 제어. |
| Express + REST | tRPC | 비-TS 클라이언트(Python · Go 에이전트)도 같은 표면을 쓰게 하려고. |

표 3은 약점이 아니라 *결정*이라는 점이 핵심이다. 트레이드오프를 *지금* 다른 방향으로 풀어 두면 V2에서 같은 코드를 다시 작성해야 한다 — 예컨대 atomic checkout을 다중 assignee로 풀면 [03 §6의 보수적 회복 규칙](03-runtime-execution.md) 전체가 바뀌고, REST를 tRPC로 바꾸면 Python/Go 에이전트의 `agent_api_keys` 흐름을 다시 그려야 한다. 표 3을 *코드의 의도된 한계*로 읽으면 어떤 새 기능이 *진짜 누락*인지 *이미 의도된 범위 밖 항목*인지를 구분할 수 있다.

## 5. 누구에게 적합한가

**적합**:

- 한 사람 / 작은 팀이 자율 에이전트 *회사* 를 운영하고 싶은 경우
- 여러 코딩 에이전트(Claude Code · Codex · Cursor 등)를 통합 거버넌스로 다루고 싶은 경우
- 토큰 비용 통제가 비즈니스 임계점인 경우
- 자기-호스팅을 받아들이는 경우 (SaaS 이용 의지 X)

**부적합**:

- *지금 당장* 매출/비용 통합 회계가 필요한 경우 (플러그인을 직접 만들거나 외부 도구 사용)
- 대규모 KB / RAG 가 핵심 가치인 경우 (Paperclip 코어가 이를 다루지 않음)
- 풀 SaaS 매니지드를 원하는 경우 (V1 은 self-hosted only)
- 멀티 멤버 보드 / 4-eye 결재가 즉시 필요한 엔터프라이즈

## 6. 성숙도 평가 — 한 줄 요약

> **Paperclip은 자율 에이전트 회사를 위한 *최소 충분* control plane이다.** 핵심 도메인(이슈 · 비용 · 거버넌스 · 어댑터 · 플러그인)이 견고히 모델링됐고, *자동화하지 않는 결정*들이 일관되게 적용되어 있다. 큰 단일 파일들과 KB/매출 영역의 부재가 향후 작업이지만, 학습용 도구이자 첫 자율 회사를 운영하기 위한 출발점으로 충분하다.

[10-research-map.md](10-research-map.md)는 10편의 리서치가 Paperclip의 어떤 모듈을 떠받치는지 매핑한다.
