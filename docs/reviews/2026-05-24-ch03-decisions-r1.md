# ch03 Round 1 — Claude Decisions

_2026-05-24, codex-a / 8 정합성 findings_

## ACCEPT — 모두 (8건)

| # | 위치 | 변경 |
|---|---|---|
| F1 | line 17 | cost event 표현 정정 — "회사·에이전트·이슈 차원 롤업" 과장 → 회사·에이전트 월간 spend 갱신, 이슈/프로젝트/run 은 귀속 축. activity.logged 도 분리 |
| F2 | line 49 | adapter.execute `heartbeat.ts` → `heartbeat.ts:7784-7808` |
| F3 | line 64 | SPEC-implementation `:421-426` → `:423-432` (Issue Status 절) |
| F4 | line 68 | execution-semantics §7 → `:155-172` (Group A 표준) |
| F5 | line 85, 90 | §8 → `:301-329`, §12 → `:449-464` + 직인용 정정 "create" → "open an" |
| F6 | line 96 | "review issue" 직인용 → 실제 "watchdog recovery action" 으로 재서술 + `:357-376` |
| F7 | line 102 | `index.ts:676-780` → `:723-760` (startup) + `:789-826` (periodic) 분리 |
| F8 | line 127 | §12 → `doc/execution-semantics.md:449-464` |

## 변경 적용 파일

- `docs/report/03-runtime-execution.md` (총 9곳 — F5 가 2곳)

## Round 2 송부 의도

Group A 표준 (execution-semantics 인용 :155-172/:174-197/:207-240, :301-329, :449-464) 본문 일관 반영. 추가 정합성 1\~2 또는 FINAL_APPROVAL 기대.
