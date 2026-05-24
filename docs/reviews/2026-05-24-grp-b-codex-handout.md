# Group B Codex Handout — 2026-05-24

## 현재 작업 중인 문제

Paperclip 학습 리포트 11챕터(`ch00`-`ch10`) Codex 합의 루프.

- 리뷰 3축: 정합성 엄격 / 표현 중도 / 품질 가볍게
- 라운드 제한: `MAX_ROUNDS=4`
- 그룹 B 범위: `ch06`-`ch10`

## 검토 대상 파일들

리포트 본문:

- `/media/restful3/data/workspace/paperclip/docs/report/00-overview.md`
- `/media/restful3/data/workspace/paperclip/docs/report/01-architecture.md`
- `/media/restful3/data/workspace/paperclip/docs/report/02-data-model.md`
- `/media/restful3/data/workspace/paperclip/docs/report/03-runtime-execution.md`
- `/media/restful3/data/workspace/paperclip/docs/report/04-adapters-and-skills.md`
- `/media/restful3/data/workspace/paperclip/docs/report/05-server-api.md`
- `/media/restful3/data/workspace/paperclip/docs/report/06-ui-and-board.md`
- `/media/restful3/data/workspace/paperclip/docs/report/07-governance-cost.md`
- `/media/restful3/data/workspace/paperclip/docs/report/08-usage.md`
- `/media/restful3/data/workspace/paperclip/docs/report/09-pros-cons.md`
- `/media/restful3/data/workspace/paperclip/docs/report/10-research-map.md`

정합성 검증 대상:

- `/media/restful3/data/workspace/paperclip/packages/`
- `/media/restful3/data/workspace/paperclip/apps/web/`
- `/media/restful3/data/workspace/paperclip/server/`
- `/media/restful3/data/workspace/paperclip/db/`

## 합의된 결정 사항

그룹 B 완료 챕터:

- `ch06` 완료
- `ch07` 완료
- `ch08` 완료, `MAX_ROUNDS` cap 종료
- `ch09` 완료, Codex `FINAL_APPROVAL` 도달

Final summary 경로:

- `docs/reviews/2026-05-24-ch06-final-summary.md`
- `docs/reviews/2026-05-24-ch07-final-summary.md`
- `docs/reviews/2026-05-24-ch08-final-summary.md`
- `docs/reviews/2026-05-24-ch09-final-summary.md`

### Cross-Chapter 표준 (Group Standard)

- DB `pgTable` 88개, `company_id` 보유 67개, schema 파일 79개
- built-in adapter 13개 (`registry.ts:513-525`)
- `packages/adapters` workspace adapter 10개 (`server/package.json:47-56` + `:71` hermes)
- 표기: `ACPX` (`acp-x` 아님), `grok-local` 포함
- `execution-semantics` 인용:
  - liveness `:155-172`
  - recovery action `:174-197`
  - stalled `:207-240`
  - stranded `:301-329`
  - watchdog silence `:357-376`
  - 보수적 회복 §11 `:405-423`
  - 결론 `:449-464`
- `SPEC.md`:
  - KB `:493`
  - revenue `:495`
  - anti-req `:519`
  - Pause Behavior `:231-240`
  - Single human Board `:24`
  - hard cap `:295-301`
- `SPEC-implementation.md`:
  - `:35` Single human board
  - `:60-71` §5.1 In Scope
  - `:67` + `:768-772` `approve_ceo_strategy` SPEC 목표
  - `:80` ClipHub Not-V1
  - `:81` multi-board
  - `:423-432` Issue Status
  - `:440-445` V1 liveness
  - `:600-615` atomic checkout
- `constants.ts` approval `:408-422`
- `agents.ts` activation `:534-548`
- `approvals.ts` `hire_agent` `:102-117`
- hard cap:
  - `budgets.ts:213-258`
  - `getInvocationBlock` `budgets.ts:716-862`
  - `cancelBudgetScopeWork` `heartbeat.ts:9579-9604`
- locale JSON 40개
- sandbox-providers 5종: `e2b`, `cloudflare`, `daytona`, `exe-dev`, `modal`

## 미해결·잔존 이견

0건

## 사용자 지시·범위 제한

- 포함: `docs/report/00~10` 본문 변경
- 제외: `docs/research/`, SVG 그림 (`DEFER`)
- 응답 파일: `docs/reviews/2026-05-24-chNN-codex-rN.md`

## 다음 단계

그룹 B 마지막 챕터:

- `ch10` — `/media/restful3/data/workspace/paperclip/docs/report/10-research-map.md`
- 목표: 4 라운드 이내 합의
