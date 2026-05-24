# ch05 Round 2 — Claude Decisions

_2026-05-24, codex-a / 3 정합성 findings (모두 ACCEPT)_

## ACCEPT (3건)

| # | 위치 | 변경 |
|---|---|---|
| F1 | line 91 | shared 타입 `cloud-upstream.ts` "zod 표현" → TypeScript type/interface (실제 파일 확인) + 라인 `:1-110` 추가 |
| F2 | KB 표기 3곳 | plugins 89→94 / agents 28→25 / environment-runtime 44→46 |
| F3 | 표 1 environments/execution-workspaces 행 | 대표 엔드포인트 정정 — `POST /projects/:pid/workspaces` 는 `projects.ts` 소속, environments/execution-workspaces 의 회사-scoped 라우트로 변경 |

## 변경 적용 파일
- `docs/report/05-server-api.md` (5곳 — F2 가 3곳)

## Round 3 송부 의도
FINAL_APPROVAL 기대.
