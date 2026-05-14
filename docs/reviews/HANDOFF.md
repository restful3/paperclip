# Paperclip 학습 리포트 합의 루프 — 핸드오프

**최종 갱신**: 2026-05-14 (후속 사이클 3 종료)
**상태**: ✅ **완료** — 본문 11 챕터 + 리서치 R01\~R10 + cross-reference + DEFER 정합화 + PDF 재빌드 전수 FINAL_APPROVAL
**다음 단계**: 신규 후속 사이클 대기 (코드 stale 검사 조건부 / 신규 챕터·리서치 추가 사용자 지시 시)

> **2026-05-14 후속 사이클 결과**: `2026-05-14-followup-cycle-final-summary.md` 참조. cross-reference 정합성 (Phase 1, substantive ACCEPT 3건: Claude billing label / Gemini adapter / SSE bridge 범위) + 그림 번호 충돌 해소 (Phase 2, Ch01·Ch03 → chapter-prefix) + PDF 재빌드 (Phase 3, 7.06 MB) 완료. 본 핸드오프의 §3 합의된 사실 묶음 + 후속 final-summary §4 가 현 시점 코드 진실.

---

## 1. 한 단락 요약

`docs/report/00-overview.md` \~ `10-research-map.md` 11개 챕터를 codex peer reviewer 로 라운드별 합의 검토했고, **전부 `===CODEX_FINAL_APPROVAL===` 종료 신호 획득**. 후속 사이클에서 `docs/research/R01\~R10` 본문 (688 LOC) 도 동일 흐름으로 합의 완료. Paperclip 코드 변경 0건. 산출물은 `docs/report/` (1,550 LOC) + `docs/research/` (688 LOC) + `docs/reviews/` (110+ 파일). 최종 결과 보고: 본문은 `docs/reviews/2026-05-14-paperclip-report-final-summary.md`, 리서치는 `docs/reviews/2026-05-14-research-cycle-final-summary.md`.

## 2. 챕터·리서치별 결과

### 2-A. 본문 챕터 (Ch00\~Ch10)

| 챕터 | 파일 (LOC) | 합의 라운드 | 누적 ACCEPT |
|---|---|---|---|
| Ch00 overview | 87 | r3 | 11+ |
| Ch01 architecture | 117 | r3 | 12+ |
| Ch02 data-model | 142 | r2 | 6 |
| Ch03 runtime-execution | 127 | r4 | 9 |
| Ch04 adapters-and-skills | 157 | r5 | 11 |
| Ch05 server-api | 180 | r4 | 10 |
| Ch06 ui-and-board | 147 | r5 | 13 |
| Ch07 governance-cost | 145 | r4 | 8 |
| Ch08 usage | 201 | r3 | 10 |
| Ch09 pros-cons | 99 | r3 | 7 |
| Ch10 research-map | 148 | r2 | 4 |

REJECT 0건. 누적 ACCEPT 약 100건. 1 챕터당 평균 3.5 라운드.

### 2-B. 리서치 본문 (R01\~R10)

| R | 파일 (LOC) | 합의 라운드 | 누적 ACCEPT |
|---|---|---|---|
| R01 agent orchestration | 62 | r2 | 5 |
| R02 Model Context Protocol | 72 | r2 | 4 |
| R03 Claude Code CLI | 88 | r3 | 5 (4+1) |
| R04 headless coding agents | 75 | r3 | 5 (4+1) |
| R05 OpenClaw / agent companies | 73 | r2 | 3 |
| R06 Drizzle / PGlite | 55 | r2 | 3 |
| R07 pnpm monorepo | 100 | r2 | 4 |
| R08 React / Vite / shadcn | 54 | r2 | 2 |
| R09 SSE / heartbeat | 57 | r2 | 5 |
| R10 LLM cost / budgeting | 53 | r2 | 2 |

REJECT 0건. 누적 ACCEPT 38건. 평균 2.2 라운드/파일.

## 3. 합의된 사실 (코드 진실 묶음)

**중요**: 이 절은 2026-05-14 시점 코드 검증 기반. 이후 코드 변경 시 stale 검사를 새 사이클 라운드 1 첫 단계로 수행.

전체 합의된 사실 묶음은 `docs/reviews/2026-05-14-paperclip-report-final-summary.md` §2 에 정리. 다음은 핵심만 추린 요약:

**어댑터**: 내장 12개 = 런타임 10 + generic 2 (`server/src/adapters/registry.ts:480-493`). 계약 2층: 장기 SPEC `invoke/status/cancel` (`doc/SPEC.md:207-215`) vs 현재 구현 `execute`+`testEnvironment` 필수 (`packages/adapter-utils/src/types.ts:349-431`). package 진입점 4개: `.` / `./server` / `./ui` / `./cli`. sandbox provider 4종.

**Data model**: pgTable 86 / 스키마 78 / `companyId` 보유 63. issue status 7개. atomic checkout `:3593-3736`. approvals 12 컬럼. activity_log actor_type 4 분류. budget hard stop 3 scope. `cost_events.agent_id` notNull.

**Runtime**: Startup reconciliation **7-pass** (`server/src/index.ts:676-780`). source-scoped recovery action. 보수적 회복 4문구: *preserve ownership / retry once / explicit recovery action / escalate visibly*. hard stop 3 메커니즘: `getInvocationBlock` / `withAgentStartLock` / `cancelBudgetScopeWork`.

**Server**: WS `authorizeUpgrade` 별도 인증. LIVE_EVENT_TYPES 9개. REST credential 5종 (board API key는 WS bearer 미지원, agent local JWT 단명).

**UI**: Vite 6 + React 19 + Tailwind 4 + shadcn/ui + TanStack Query 5 + router 7. `/dashboard/live` = `ActiveAgentsPanel` (활성/최근 run). 첫 화면 입력 2 항목 (Company name + Mission/goal). `/agents/new` 입력 7 항목.

**호스팅**: Docker compose 풀스택 (db + server). 단독 `docker run` 도 `BETTER_AUTH_SECRET` 필수. authenticated/public 4 env 필수. board-claim 조건부 흐름.

**Workspace**: pnpm workspace 24 패키지 (adapters 9 · plugins · server · ui · cli). 회사 멤버십 + 휴먼 role 4단계. multi-member boards = Not V1.

## 4. DEFER 보존 항목 (다음 사이클에서 일괄 정합화 후보)

- 라인 범위 ±1\~2줄 nit
- 영문 용어 취향 (org tree · Augmented LLM 등)
- 캡션 문체 (코드/그림 N. 부제 형식)
- **외부 인용 그림 글로벌 번호 정합화** — 챕터 간 같은 번호 충돌 (예: Ch01·Ch02의 "그림 2")
- 거대 단일 파일 분해 (`services/company-portability.ts` 180 KB 등) — Paperclip 코드 영역, 리포트 범위 밖

## 5. 운영 관습 (재사용 가능)

**라운드 흐름**:
1. Claude → Codex: ChNN rN 요청 (검토 대상 + 진실 소스 + 합의된 사실 인용)
2. Codex → ChNN-codex-rN.md 응답
3. Claude: ACCEPT 항목 코드 대조 검증 → decisions-rN.md
4. Claude: 본문 수정 → changes-rN.md
5. Claude → Codex: ChNN r(N+1) 재리뷰 요청
6. Codex: 새 substantive 사실 오류 없으면 첫 줄 `===CODEX_FINAL_APPROVAL===`

**평가 기준 5개**: 사실 정확성 (`path:line`) / 표현 효율성 / 한국어 자연성 / 마크다운 관습 / 인용 완결성.

**마크다운 관습**:
- CJK bold 공백: `**...**` 내부 구두점 + 닫는 `**` 뒤 한글 시 공백 1칸 또는 `<strong>`
- 캡션 `**그림 N. 〈제목〉**`, 본문 그림 참조는 평문
- 코드 펜스에 언어 식별자
- 본문 `~` 는 `\~` 이스케이프 (HTML 블록 내부는 raw)

**nit 자제 정책**: 라인 범위 ±1\~2 nit 는 DEFER. substantive 사실 오류 위주 ACCEPT.

**tmux/codex 환경**:
- TMUX_TARGET = `paperclip:codex` (또는 현재 세션의 codex 윈도우)
- `mktemp` → `tmux load-buffer` → `paste-buffer` → Enter (안 먹으면 한 번 더, Escape는 위험)
- 폴링: `/home/restful3/.claude/skills/codex-peer-reviewer/scripts/poll_codex.sh paperclip:codex 1800 30 5` (백그라운드, exit 2여도 응답 파일은 정상)
- handout-then-clear: Context ≥ 85% 임계. 단 Codex가 자체 압축 자주 함 → 실제로 거의 발동 안 됨.

## 6. 사용자 협업 메모

- 자율 합의 루프 모드 (중간 보고 없이 final-summary 까지 일괄 진행) 선호
- 라운드 캡 없음 — `===CODEX_FINAL_APPROVAL===` 만 종료 신호
- Self-termination: 2 라운드 연속 0 ACCEPT 시 (이번 사이클 미발생)
- 폴링 대기 narration 최소화

## 7. 후속 사이클 후보

1. ~~챕터 ↔ 리서치 cross-reference 일관성~~ — ✅ **완료** (2026-05-14 후속 Phase 1, substantive ACCEPT 3건)
2. ~~DEFER 일괄 정합화~~ — ✅ **완료** (2026-05-14 후속 Phase 2, 그림 충돌 3쌍 해소, R09 `~` 점검)
3. ~~PDF 재빌드~~ — ✅ **완료** (2026-05-14 후속 Phase 3, 7.06 MB)
4. **Paperclip 코드 변경 후 리포트·리서치 stale 검사** — §3 합의 사실들 + 후속 final-summary §4 의 검증 일자 (2026-05-14) 이후 코드 변경 발생 시.
5. **신규 챕터 / 리서치 추가** — 예: 플러그인 SDK 심층, sandbox provider 4종 비교, 어댑터 12개 마이그레이션 가이드, cost_events 시간-기반 modifier (Gemini storage 단가 등) 확장 — 사용자가 도메인 지정 시.

## 8. 새 세션 시작 프롬프트 (그대로 입력)

후속 사이클 시작 시 사용. 어떤 사이클인지 마지막 줄에서 골라 지정:

> Paperclip 학습 리포트 합의 루프 후속 작업을 진행해줘. 이전 사이클은 모두 종료된 상태야 — 핸드오프는 `/media/restful3/data/workspace/paperclip/docs/reviews/HANDOFF.md` 에 있고, 합의된 사실 묶음 (§3) · 운영 관습 (§5) · DEFER 항목 (§4) · 후속 사이클 후보 (§7) 를 그대로 따른다. codex peer reviewer 스킬 사용, tmux 타깃 `paperclip:codex`, 자율 모드 (중간 보고 없이 끝나면 일괄 보고). 진행할 작업: [§7 의 후보 중 하나 선택, 또는 새 지시].

---

**이전 사이클 산출물 빠른 인덱스**

```
docs/report/
├── 00-overview.md          ✅ Ch00 합의
├── 01-architecture.md      ✅ Ch01 합의
├── 02-data-model.md        ✅ Ch02 합의
├── 03-runtime-execution.md ✅ Ch03 합의
├── 04-adapters-and-skills.md ✅ Ch04 합의
├── 05-server-api.md        ✅ Ch05 합의
├── 06-ui-and-board.md      ✅ Ch06 합의
├── 07-governance-cost.md   ✅ Ch07 합의
├── 08-usage.md             ✅ Ch08 합의
├── 09-pros-cons.md         ✅ Ch09 합의
└── 10-research-map.md      ✅ Ch10 합의

docs/research/
├── 01-agent-orchestration-control-plane.md  ✅ R01 합의
├── 02-model-context-protocol.md             ✅ R02 합의
├── 03-claude-code-cli.md                    ✅ R03 합의
├── 04-headless-coding-agents.md             ✅ R04 합의
├── 05-openclaw-agent-companies.md           ✅ R05 합의
├── 06-drizzle-pglite.md                     ✅ R06 합의
├── 07-pnpm-monorepo.md                      ✅ R07 합의
├── 08-react-vite-shadcn.md                  ✅ R08 합의
├── 09-sse-heartbeat-patterns.md             ✅ R09 합의
└── 10-llm-cost-budgeting.md                 ✅ R10 합의

docs/reviews/
├── 2026-05-14-paperclip-report-final-summary.md   ⭐ 본문 사이클 결과
├── 2026-05-14-research-cycle-final-summary.md     ⭐ 리서치 사이클 결과
├── 2026-05-14-paperclip-report-codex-handout.md
├── 2026-05-14-research-cycle-codex-handout.md
├── 2026-05-14-ch00 \~ ch10 -{decisions,changes,codex}-r{1..5}.md  (64 파일)
├── 2026-05-14-r01 \~ r10 -{decisions,changes,codex}-r{1..3}.md   (46 파일)
└── HANDOFF.md (이 파일)
```
