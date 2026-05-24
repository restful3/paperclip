# ch08 Final Summary

_2026-05-24, codex-b / 4 rounds / **MAX_ROUNDS cap** 종료_

## Outcome: **capped at MAX_ROUNDS** (R4 의 추가 finding ACCEPT 후 cap)

ch00 과 동일 패턴 — Codex 가 매 라운드 새 정합성 1\~2개 발견. R4 응답까지 ACCEPT 한 뒤 R5 송부하지 않고 종료.

## 변경 통계

| 라운드 | finding | ACCEPT | LOC |
|---|---|---|---|
| R1 | 8 정합성 | 8 | ~25 |
| R2 | 3 정합성 | 3 | ~3 |
| R3 | 2 정합성 | 2 | ~3 |
| R4 | 1 정합성 | 1 | 1 |
| **합** | **14** | **14** | **~32** |

## 본문 변경 영역

`docs/report/08-usage.md`:
- line 47: `--bind lan` 주석에 "첫 실행(설정 없음) 때" 추가
- line 73: 예산 API endpoint vs route-local 정의 분리
- line 96: NewAgent — AgentConfigForm 의 default env/adapter config/run policy 명시
- line 100: CEO 자기 보고 라인 *제안* (SPEC 목표) → 현재 구현 흐름
- line 106: 보드 관리 범위 정정 (CEO 한 명만 → 전체 org tree 직접 관리 가능)
- line 112-114: CEO 첫 heartbeat 전략 분해 4단계 → 현재 구현 흐름
- line 167: Dockerfile `:76` → `:79-80` + docker-compose.yml `:2-16, 18-31`
- line 192: board-claim.ts 라인 정정 + routes/access.ts 추가
- line 198: /dashboard/live — live-runs API + run log polling (server route + UI API base 인용)

## DEFER

- `docs/research/06-drizzle-pglite.md` 등 research scope 별도 작업
