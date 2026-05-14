# Paperclip 학습 리포트 — 리서치 사이클 (R01\~R10) 최종 요약

**날짜**: 2026-05-14
**상태**: ✅ **완료** — 10 개 리서치 파일 전수 `===CODEX_FINAL_APPROVAL===`
**선행 사이클**: `2026-05-14-paperclip-report-final-summary.md` (Ch00\~Ch10 본문 합의)

---

## 1. 한 단락 요약

`docs/research/` 산하 R01\~R10 (총 688 줄) 리서치 본문을 codex peer reviewer 로 합의 루프 검증했다. **전부 `===CODEX_FINAL_APPROVAL===` 종료 신호 획득**. 본문 (688 줄) 외 산출은 `docs/reviews/` 의 합의 사이클 파일 46 개 (decisions / changes / codex r1\~r3 × 10 파일) + 본 요약 + handout. Paperclip 코드 변경 0 건.

## 2. 파일별 결과

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
| R09 SSE / heartbeat patterns | 57 | r2 | 5 |
| R10 LLM cost / budgeting | 53 | r2 | 2 |

REJECT 0 건. DEFER 0 건 (substantive only 정책). 누적 ACCEPT **38 건**. 평균 2.2 라운드/파일.

## 3. 핵심 발견 (substantive 사실 오류 패턴)

### 3-A. 직접 quote 검증 실패가 가장 빈번
- R01: Anthropic workflows/agents · Temporal Event History · LangGraph quote · n8n quote 4 건 모두 원문 truncation 또는 부재.
- R02: ChatGPT MCP 도입 specifics (2025-09-10, 2025-03 채택) InfoQ 미지원.
- R03: "headless mode 로 불렸으며" 표현 부재. Agent SDK 개명 문장도 overview 미지원.
- R04: Cursor blog (2025-08-07) "January 2026" / Background Agents API 미언급.
- R09: Anthropic "operate autonomously over multiple turns" 표현 부재.
- R10: LangSmith cost 산정은 cost-tracking 페이지 (observability-concepts 아님).

→ **교훈**: 직접 quote는 WebFetch 원문 확인 없이는 신뢰 불가. paraphrase 가 더 안전.

### 3-B. 수치 stale / 미지원
- R02: Zylos server 수 "1,200 → 9,400+" → 실제 18,000+ (Glama 18,374 / MCP.so 18,250+).
- R06: Paperclip 테이블 수 "75 개 안팎" → 86 개 pgTable / 78 개 스키마 파일 (Ch02 합의).
- R10: Gemini context caching storage $4.50/1M/hour 단가 누락.

### 3-C. Paperclip 코드 stale fact / 모호한 일반화
- R01: budget hard-stop 추적축 (8 개) ≠ 집행 scope (3 개) 혼동.
- R02: 어댑터 = MCP client 혼동. Ch04 §"수직 직교" 합의 사실 누락.
- R03: parse.ts 3 신호 (`extractClaudeRetryNotBefore` / `isClaudeTransientUpstreamError` / `detectClaudeLoginRequired`) 단일 retry 결정으로 묶음.
- R04: 어댑터 공통 보고는 usage/cost/session 만 — tool 정보는 pi-local·opencode-local 만 보조.
- R05: 어댑터 목록에 cursor-cloud 누락 (정정으로 9 개 알파벳순).
- R06: 마이그레이션 결정 — `drizzle.config.ts` vs `resolveMigrationConnection()` 혼동.
- R07: 워크스페이스 글롭 7 entry + 2 exclude, 패키지명 `@paperclipai/server`, overrides 위치 (pnpm 9 root json).
- R08: TanStack Query 훅은 `api/` 가 아닌 `context/` + pages/components 분산. CSRF 코드 부재.
- R09: `adapter.execute(ctx)` vs eventType `adapter.invoke`, silent-run 신호는 `lastOutputAt`, 경로 `doc/execution-semantics.md`.

### 3-D. 표준 강도 / 시점
- R02: Streamable HTTP `Origin` 검증 MUST, localhost 바인딩 SHOULD — 강도 분리.
- R09: SSE 지수 백오프 may, not must.
- R05: company import CLI — positional `<fromPathOrUrl>`, catalog README 의 `--from` 옵션 예시와 차이.

## 4. 운영 관습 재확인

이전 사이클의 라운드 흐름 (`HANDOFF.md §5`) 그대로 작동했다. 추가 관찰:

- **직접 quote 위험**: 직접 인용은 truncation·창작·stale 의 3 위험을 동시에 안음. 다음 사이클부터 직접 quote 사용 시 본 사이클의 4 단 검증 (WebFetch + 정확 매칭) 의무화 권장.
- **paste-buffer Enter 횟수**: codex CLI 가 multiline 메시지를 받을 때 Enter 1\~3 회 가변. capture-pane 으로 "Working" 상태 확인 후 폴링 시작이 안정.
- **ScheduleWakeup 60\~120s** 와 백그라운드 폴링 병용으로 codex 응답 stall 방지. 사용자 30 초 강제 체크 요청이 효과적이었음.
- **handout-then-clear**: 본 사이클에서 R04 r2 시점 context 87% → handout 작성 후 9% 까지 codex 자체 압축. 명시적 `/clear` 없이도 보존 가능 (codex 자체 압축이 충분히 강함을 시사).

## 5. 변경 인벤토리 (코드 변경 0 건)

```
docs/research/
├── 01-agent-orchestration-control-plane.md  ✅ 5 라인 수정
├── 02-model-context-protocol.md             ✅ 4 라인 수정
├── 03-claude-code-cli.md                    ✅ 5 라인 수정 (r1 4 + r2 1)
├── 04-headless-coding-agents.md             ✅ 5 라인 수정 (r1 4 + r2 1)
├── 05-openclaw-agent-companies.md           ✅ 3 라인 수정
├── 06-drizzle-pglite.md                     ✅ 3 라인 수정
├── 07-pnpm-monorepo.md                      ✅ 3 라인 + 1 substring 수정
├── 08-react-vite-shadcn.md                  ✅ 2 라인 수정
├── 09-sse-heartbeat-patterns.md             ✅ 4 라인 수정
└── 10-llm-cost-budgeting.md                 ✅ 3 라인 수정

docs/reviews/
├── 2026-05-14-research-cycle-final-summary.md  ⭐ 본 파일
├── 2026-05-14-research-cycle-codex-handout.md  ⭐ 중간 handout
├── 2026-05-14-r01-decisions-r1.md
├── 2026-05-14-r01-changes-r1.md
├── 2026-05-14-r01-codex-r1.md
├── 2026-05-14-r01-codex-r2.md  (FINAL_APPROVAL)
├── ...  (R02~R10 동일 패턴, 총 46 파일)
└── 2026-05-14-r10-codex-r2.md  (FINAL_APPROVAL)
```

## 6. 잔존 항목 / 후속 사이클 후보

### 6-A. 즉시 정합화 후보 (DEFER 일괄 정리)
- R09 본문에 코드 펜스 없이 `~` 가 본문에 직접 등장하는 곳 (k=2~3 등) — markdown 렌더링 시 strikethrough 트리거 위험. 일괄 \~ 이스케이프 점검 권장.
- 외부 인용 그림 글로벌 번호 정합화 (이전 사이클부터 미해결 DEFER).

### 6-B. 다음 사이클 후보
1. **챕터 본문 ↔ 리서치 본문 cross-reference 통합 점검** — R02 §5, R03 §3, R04 §4, R09 §5 등이 챕터(Ch01·Ch04·Ch03·Ch05)와 일관성 맺어야 하는데 본 사이클은 단일 파일별 검증만 했다. 챕터-리서치 짝 검증으로 미발견 일관성 오류 잡을 수 있음.
2. **Paperclip 코드 변경 후 리포트·리서치 stale 검사** — 2026-05-14 검증일 이후 변경 시.
3. **신규 리서치 챕터 추가** — 본 사이클에서 codex 가 발견한 빈 영역 (예: Gemini context caching storage 단가 같은 시간 기반 modifier 가 cost_events 스키마에 반영 안 됐다는 발견은 향후 schema 확장의 후보).
4. **PDF 재빌드** — `docs/report/pdf/` 의 6.5MB PDF는 합의 전 버전일 수 있음. 본문 변경 후 재컴파일.

## 7. 다음 세션 진입 프롬프트

> Paperclip 학습 리포트 후속 작업을 진행해줘. 리서치 본문 사이클은 모두 종료됐고, 핸드오프는 `/media/restful3/data/workspace/paperclip/docs/reviews/HANDOFF.md` 에 정리됐다. 합의된 사실 묶음 (§3) · 운영 관습 (§5) · 후속 사이클 후보 (§7) 따라 다음 작업 진행: [§6-B 의 후보 중 하나 또는 신규 지시].
