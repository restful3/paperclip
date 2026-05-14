# 챕터 02 — 라운드 1 결정 로그

검증 결과 모두 ACCEPT (9건). DEFER 2건 수용. OUT-OF-SCOPE 1건 수용.

## ACCEPT (9)

| # | 위치 | 검증된 사실 |
|---|---|---|
| A1 | `:11` | pgTable 정의 86개, 스키마 파일 78개 (`schema/index.ts:1-78` + `grep -c pgTable schema/*.ts` = 86) |
| A2 | `:17,33` | `issues.checkout_run_id`/`execution_run_id` → `heartbeat_runs.id` 직접 FK (`packages/db/src/schema/issues.ts:37-38`) |
| A3 | `:17,32` | `cost_events.agent_id` notNull, 나머지 4개(`issue_id`/`project_id`/`goal_id`/`heartbeat_run_id`) nullable (`packages/db/src/schema/cost_events.ts:13-18`) |
| A4 | svg 내부 라벨 | OUT-OF-SCOPE로 격하 — svg 수정은 별도 그림 작업, 본문 텍스트에서 해당 라벨 직접 인용 없음 |
| A5 | `:52,60` | atomic checkout 코드 위치 `server/src/services/issues.ts:3593-3736`. status `open` 없음, 7개 = backlog/todo/in_progress/in_review/done/blocked/cancelled (`packages/shared/src/constants.ts:127-135`) |
| A6 | `:93,108` | 마이그레이션 출력 경로 `./src/migrations` (`packages/db/drizzle.config.ts:5`), 실제 파일도 `packages/db/src/migrations/0000_...0084_...sql`. "자동 적용 없다"는 표현은 보강 (dev/server boot 경로에 pending migration 자동 적용 조건이 있음) |
| A7 | `:132-134` | Supabase pooler 6543 vs direct 5432 분리 — Codex 제안대로 두 줄 예시 |
| A8 | `:87` | 실제 테이블명: `plugin_database_namespaces`, `plugin_migrations`, `plugin_state` (`schema/index.ts:73-75`). `plugin_database`는 *파일명*이지 테이블명 아님 |
| A9 | `:110,120` | 외부 인용 그림 메인 번호 통합 — `그림 4-A` → `그림 5`, `그림 4-B` → `그림 6` (Ch00 합의 적용, handout §합의 사항) |

## DEFER (2) — 수용

- `:21` "모든 mutating call" 권한 검사 — API 장(Ch05)에서 별도 검증
- `:37` prompt caching 가격 해석 — 비용 챕터(Ch07) 또는 research 10에서 인용 밀도 조정

## OUT-OF-SCOPE — 수용

- `_pdf-build` 산출물 재생성 — 본문 합의 후 일괄
- `assets/fig-04-er-core.svg` 라벨 정정 — 별도 그림 작업 (A4 격하)

## 누적 LOC

- Ch02 r1 예상 변경: ~25 LOC. 전체 누적: Ch00 ~32 + Ch01 ~40 + Ch02 r1 ~25 = ~97 / 200 checkpoint.
