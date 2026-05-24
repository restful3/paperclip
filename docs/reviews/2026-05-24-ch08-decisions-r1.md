# ch08 Round 1 — Claude Decisions

_2026-05-24, codex-b / 8 정합성 findings_

## ACCEPT — 모두 (8건)

| # | 위치 | 변경 |
|---|---|---|
| F1 | line 47 | `run --bind lan` 주석에 "첫 실행(설정 없음) 때" 추가 |
| F2 | line 73 | `PATCH /companies/...` → `PATCH /api/companies/...` + `companies.ts:140-149` → `:117-150` |
| F3 | line 96 | NewAgent 표현 완화 — AgentConfigForm 의 default environment / adapter config / run policy 도 함께 받는다 명시 |
| F4 | line 100 | CEO 자기 보고 라인 *제안* (SPEC 목표) → 현재 구현 흐름 (각 hire 가 hire_agent approval) |
| F5 | lines 112-114 | CEO 첫 heartbeat 전략 분해 4단계 → 현재 구현 흐름 (보드/사용자가 goal/issue 생성, approve_ceo_strategy 는 SPEC 목표) |
| F6 | line 167 | `Dockerfile:76` → `:79-80` |
| F7 | line 192 | `board-claim.ts:43-66` → `:44-73` + `routes/access.ts:2454-2495` |
| F8 | line 198 | `/dashboard/live` WebSocket 위 실시간 → live-runs API + run log polling 으로 정정 |

## 변경 적용 파일
- `docs/report/08-usage.md` (8곳)

## Round 2 송부 의도

cross-chapter 정합 (approve_ceo_strategy SPEC 목표로 ch07 와 정렬). FINAL_APPROVAL 또는 미세 정합 1\~2 예상.
