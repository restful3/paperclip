# ch04 Round 1 — Claude Decisions

_2026-05-24, codex-a / 8 정합성 findings (모두 ACCEPT)_

## ACCEPT — 모두 (8건)

| # | 위치 | 변경 |
|---|---|---|
| F1 | line 13 | `registry.ts:511-524` → `:513-525` |
| F2 | line 19 | local coding adapters 목록에 `grok-local` 추가 + `:128-138, 488` → `:497` |
| F3 | line 25, 33-34 | 표 헤더 "10종" → "11종", `gemini-local` 다음에 `grok-local` 행 추가 |
| F4 | line 31-32 | Cursor 라이선스/요금 — BYOK + Cursor subscription/usage billing |
| F5 | line 42 | 라이선스 해석 — grok-local 추가, ACP → ACPX (Group A 표준) |
| F6 | line 107 | Hermes 표현 — package.json:71 + registry.ts:128-138 + :523 분리 |
| F7 | line 111 | sandbox-providers 4종 → 5종 (modal 추가) |
| F8 | line 141 | plugin-managed-skills 인용 라인 정정 (:247-268 + :640-648 + :737-745 + sdk index :286-299 + types :919-929) |

## 변경 적용 파일
- `docs/report/04-adapters-and-skills.md` (9곳 — F3 표 row 추가 포함)

## Round 2 송부 의도
Group A 표준 (built-in 13개 / packages/adapters 10개 / ACPX) 일관 반영. Hermes 분리 표준도 정렬. FINAL_APPROVAL 또는 미세 정합 1\~2 예상.
