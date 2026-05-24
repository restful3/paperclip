# ch07 Round 1 — Claude Decisions

_2026-05-24, codex-b / 10 정합성 findings 분류 / 표현·품질 이상 없음_

## ACCEPT — 모두 (10건)

Codex 가 모든 지적을 코드 라인 인용으로 자기 검증. 큰 변경 2건(F4 approve_ceo_strategy 표현 낮춤, F5 라우트 행 재정렬)을 포함.

| # | 위치 | 변경 요약 |
|---|---|---|
| F1 | line 16 | SPEC §10 → SPEC-implementation:35 / SPEC.md:24 (절번호 → 라인 인용) |
| F2 | line 38 | constants.ts approval :373-388 → :408-422 |
| F3 | line 42 | activation 라인: approvals.ts:102-117 + agents.ts:534-548 함께 인용 |
| F4 | line 43 | approve_ceo_strategy 현재 구현 근거 없음 → SPEC 목표로 표현 낮춤 (SPEC-implementation.md:67, :768-772 인용) |
| F5 | line 60 | 결정 override 라우트 `POST /api/issues/:id/decisions` → `PATCH /api/issues/:id` + watchdog 분리 표기 |
| F6 | line 83 | 비용 normalization: heartbeat.ts:1299-1302 → :1358-1361 + :6764-6803 createEvent, API 보고는 validators/cost.ts:4-23 + routes/costs.ts:73-85 분리 |
| F7 | line 83 | budget_incidents "실제 컬럼" → "핵심 컬럼" + 추가 컬럼 명시 |
| F8 | line 93/95 | hard cap 라인 범위: budgets.ts :213-253 → :213-258, heartbeat.ts :9431-9456 → :9579-9604 cancelBudgetScopeWork() + helper :9431-9477 |
| F9 | line 113 | costs.byProject :454~492 → :454-502 |
| F10 | line 136 | activity_log cleanup: agents.ts :501-516 → :501-518, companies.ts :265-270 → :267-283 |

## 변경 적용 파일

- `docs/report/07-governance-cost.md` (10곳 수정)

## Round 2 송부 의도

10건 모두 ACCEPT. 코드 라인 정합 + 약간의 표현 표준화. FINAL_APPROVAL 또는 추가 미세 정합 1\~2개 예상.
