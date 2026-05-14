# 챕터 05 — 라운드 1 결정 로그

## ACCEPT (6)
| # | 위치 | 사실 |
|---|---|---|
| A1 | `:19, 89-103` | WebSocket은 HTTP upgrade handler. 인증은 actorMiddleware 재사용이 아니라 별도 authorizeUpgrade — local_trusted/Better Auth/agent API key |
| A2 | `:107, 120` | LIVE_EVENT_TYPES 9개 (`packages/shared/src/constants.ts:596-606` 검증) — `plugin.worker.crashed`/`restarted` 둘이 별개 type |
| A3 | `:40` | secrets 라우트 확장: provider listing/configs/remote import/rotate/access-events 등 |
| A4 | `:31, 44` | recovery-actions API + `issue-recovery-actions.ts` 서비스 명시 |
| A5 | `:42` | `org-chart-svg.ts`는 renderer helper. 실제 endpoint는 `agents.ts` 안 `org.svg`/`org.png` |
| A6 | `:152-166` | plugin hook 카테고리에 managed agents/projects/routines/**skills** 추가 — `plugin-managed-{skills,agents,routines}.ts` 검증됨 |

## DEFER 2 / OUT-OF-SCOPE 1 — 수용

## 누적 LOC
- Ch05 r1 예상 변경: ~25 LOC. 전체 누적: ~210 / 200. **체크포인트 초과** — Codex final approval 후 final-summary 단계에서 사용자 보고
