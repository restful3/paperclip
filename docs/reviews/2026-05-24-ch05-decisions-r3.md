# ch05 Round 3 — Claude Decisions

_2026-05-24, codex-a / 1 정합성 finding (ACCEPT)_

## ACCEPT

### F1 — line 29 (표 1 companies.ts 행): `:id` → `:companyId`
- **근거**: 실제 라우트 `router.patch("/:companyId", ...)` (`server/src/routes/companies.ts:329`). app.ts:208 에서 `/companies` 아래 mount. Round 2 의 `projects.ts` 정정과 일관성.

## 변경 적용 파일
- `docs/report/05-server-api.md` (1곳)

## Round 4 송부 의도
MAX_ROUNDS 진입 — FINAL_APPROVAL 또는 cap 종료.
