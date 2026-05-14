# 챕터 03 — 라운드 1 결정 로그

## ACCEPT (9)

| # | 위치 | 검증된 사실 |
|---|---|---|
| A1 | `:29-43` | 어댑터 계약 SPEC vs 구현 두 층 분리 — 장기 SPEC `invoke/status/cancel` (`doc/SPEC.md:207-215`) vs 구현 `execute/testEnvironment` (`packages/adapter-utils/src/types.ts:349-352`) |
| A2 | `:11, 17` | heartbeat 흐름에서 checkout 주체는 server-side `issuesSvc.checkout`이고 그 뒤 `adapter.execute` 호출. 흐름 정정 |
| A3 | `:17, 72-75` | t₆ "락 해제" → `execution_run_id` 만 해제, `checkout_run_id` 는 별도. 표 1과 본문 정밀화 |
| A4 | `:55-62` | 상태 전이 표 정정 — terminal은 `done`/`cancelled`, "어떤 상태든 → blocked/cancelled" 단정 제거, blocked 자동 in_progress 전이는 *wake → checkout 성공시* |
| A5 | `:62, 84` | "explicit recovery issue" → "explicit recovery action" (`issue_recovery_actions` 테이블 모델, `packages/db/src/schema/issue_recovery_actions.ts:16-67`) |
| A6 | `:77-85` | stranded recovery 2단계 확장 — 한 번 retry 후 `blocked` + source-scoped recovery action upsert + owner/wake policy |
| A7 | `:86-92` | silent watchdog → recovery action으로 표면화, evaluation issue는 issue-backed 구현 형태로 |
| A8 | `:94-107` | startup 4-pass → 7-pass (코드 2를 "핵심 4-pass"로 좁히거나 실제 순서로 갱신) |
| A9 | `:113-115` | `status()` SPEC 표현 → 현재 구현 표면(`heartbeat_runs.status`, watchdog evidence 등)으로 |

## DEFER (2) — 일부 수용

- D1. 그림 5-A 외부 인용 그림 번호 통합 — *글로벌 번호 정책*이 챕터 간에 깨져 있는 게 발견되어, 모든 챕터 합의 후 final-summary 단계에서 일괄 정합화 (Ch00·Ch01·Ch02 본문도 같이 손볼 것)
- D2. "단조 증가" 다이어그램 문장 — 다음 라운드에 표현 효율성 정리 단계로

## OUT-OF-SCOPE
- svg 라벨 수정 (`fig-05-heartbeat-seq.svg`, `fig-06-task-state.svg`) — 별도 그림 작업

## 누적 LOC
- Ch03 r1 예상 변경: ~35 LOC (큰 §3, §6, §7 단락 재작성). 전체 누적: ~135 / 200 checkpoint. **임박 — Ch04 시작 전 사용자 보고 필요**
