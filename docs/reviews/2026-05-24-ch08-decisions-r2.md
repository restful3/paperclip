# ch08 Round 2 — Claude Decisions

_2026-05-24, codex-b / 3 정합성 findings (모두 ACCEPT)_

## ACCEPT (3건)

| # | 위치 | 변경 |
|---|---|---|
| F1 | line 73 | route-local 정의 + `/api` mount 분리 표기 (`app.ts:196-224, 314-317` 추가) |
| F2 | line 167 | docker-compose.yml :18-31 → :2-16, 18-31 (db + server 두 서비스) |
| F3 | line 198 | `/dashboard/live` 인용 보강 — `ActiveAgentsPanel.tsx:78-81, 109-116`, `heartbeats.ts:108-120`, `useLiveRunTranscripts.ts:250-259` |

## 변경 적용 파일
- `docs/report/08-usage.md` (3곳)

## Round 3 송부 의도
FINAL_APPROVAL 기대 — Round 1 의 큰 변경(F4, F5 SPEC 목표 정렬) 모두 정합 OK 확인 후 미세 라인 정합만 남았던 상태.
