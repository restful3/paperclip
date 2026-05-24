# ch09 Round 1 — Claude Decisions

_2026-05-24, codex-b / 7 정합성 findings (모두 ACCEPT)_

## ACCEPT (7건)

| # | 위치 | 변경 |
|---|---|---|
| F1 | line 29 | execution-semantics §12 :407-422 → §11 :405-423, SPEC-implementation :434-442 → :440-445 |
| F2 | line 31 | hard ceiling 라인 — getInvocationBlock 정의는 budgets.ts:716-862, 호출은 heartbeat.ts:3976-3982/:8758, cancelBudgetScopeWork :9431-9456 → :9579-9604, SPEC.md:295-300 → :295-301 |
| F3 | line 33 | activity_log 불변 → append 중심 + live-events-ws.ts:63-65, 208-211 인용 |
| F4 | line 46 | IssueDetail 경로 `ui/pages/` → `ui/src/pages/` + 163 KB → 164 KB |
| F5 | line 48 | SPEC.md :489-493 → :491-495 + SPEC-implementation.md:81 추가 |
| F6 | line 53 | Inbox.tsx 101 KB → 108 KiB + README.md:123-124 + 경로 `ui/src/pages/` |
| F7 | line 55 | "ClipHub coming soon" 직접 인용 → SPEC-implementation.md:80 + CLIPHUB.md:1-13, 204-213 |

## 변경 적용 파일
- `docs/report/09-pros-cons.md` (7곳)

## Round 2 송부 의도
FINAL_APPROVAL 기대.
