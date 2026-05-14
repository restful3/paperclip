# 챕터 04 — 라운드 1 결정 로그

CHANGE-IMPACT STALE 판정대로 5건 ACCEPT. DEFER 2 / OUT-OF-SCOPE 2.

## ACCEPT (5)

| # | 위치 | 검증된 사실 |
|---|---|---|
| A1 | `:13, 15, 25` + 그림 7 | "런타임 9 + generic 2 = 11" → "런타임 10 + generic 2 = 12" (cursor-cloud 신규, `server/src/adapters/registry.ts:480-493`, `packages/adapters/cursor-cloud/`) |
| A2 | `:71, 85, 145` | `invoke/status/cancel` → `execute/testEnvironment` (Ch03 합의 적용) |
| A3 | `:91-106` 후 | sandbox provider plugin 4종(`e2b` 기존 + `cloudflare`/`daytona`/`exe-dev` 신규)을 §4.5 또는 §6에 추가 — `packages/plugins/sandbox-providers/` 확인 |
| A4 | `:108-133` 내 | plugin-managed-skills(`server/src/services/plugin-managed-skills.ts:1-359` 검증됨) + LLM Wiki 예시 단락 §5에 추가 |
| A5 | `:118-131` | "6개 추가 스킬" → "7개" + `para-memory-files/` 추가 (`skills/` 8 디렉토리 확인) |

## DEFER (2) — 수용
- 외부 인용 그림 `4-1~4` 번호 통합 — 글로벌 정합 단계
- `server/skills/` 잘못된 경로 — 본문은 root `skills/` 사용 중 (정확)

## OUT-OF-SCOPE (2) — 수용
- `assets/fig-07-adapter-matrix.svg` 재배치
- PDF 재빌드
