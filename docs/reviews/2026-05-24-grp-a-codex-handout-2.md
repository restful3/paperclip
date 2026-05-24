# Group A Codex Handout 2 — 2026-05-24

## 현재 작업 중인 문제

Paperclip 학습 리포트 11챕터(`ch00`-`ch10`)의 Codex 합의 루프를 진행 중이다.

- 그룹 A: `ch00`-`ch05`
- 그룹 B: `ch06`-`ch10`
- 리뷰 3축:
  - 정합성: 엄격
  - 표현: 중도
  - 품질: 가볍게
- 챕터당 `MAX_ROUNDS=4`

## 검토 대상 파일들

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

정합성 검증:

- `/media/restful3/data/workspace/paperclip/packages/`
- `/media/restful3/data/workspace/paperclip/apps/web/`
- `/media/restful3/data/workspace/paperclip/server/`
- `/media/restful3/data/workspace/paperclip/db/`

## 합의된 결정 사항

그룹 A 완료 챕터:

- `ch00` 완료
  - final summary: `/media/restful3/data/workspace/paperclip/docs/reviews/2026-05-24-ch00-final-summary.md`
- `ch01` 완료
  - final summary: `/media/restful3/data/workspace/paperclip/docs/reviews/2026-05-24-ch01-final-summary.md`
- `ch02` 완료
  - final summary: `/media/restful3/data/workspace/paperclip/docs/reviews/2026-05-24-ch02-final-summary.md`
- `ch03` 완료
  - final summary: `/media/restful3/data/workspace/paperclip/docs/reviews/2026-05-24-ch03-final-summary.md`
- `ch04` 완료: `FINAL_APPROVAL`
  - final summary: `/media/restful3/data/workspace/paperclip/docs/reviews/2026-05-24-ch04-final-summary.md`

### Cross-Chapter 표준 (Group A standard)

- DB `pgTable`: 88개
- `company_id` 보유 테이블: 67개
- schema 파일: 79개
- built-in adapter: 13개
  - 기준: `server/src/adapters/registry.ts:513-525`
- `packages/adapters` workspace adapter: 10개
  - 기준: `server/package.json:47-56`
  - Hermes npm dependency: `server/package.json:71`
- 표기: `ACPX`
  - `acp-x`, `ACP-x` 아님
  - `grok-local` 포함
- `doc/execution-semantics.md` 인용 기준:
  - liveness contract: `:155-172`
  - explicit recovery action: `:174-197`
  - stalled 신호: `:207-240`
  - stranded patterns: `:301-329`
  - watchdog silence: `:357-376`
  - 보수적 회복 결론: `:449-464`
- `doc/SPEC.md` 인용 기준:
  - KB future plugin: `:493`
  - revenue/expense future plugin: `:495`
  - revenue/expense anti-requirement: `:519`
  - Pause Behavior: `:231-240`
  - Single human Board: `:24`
- `doc/SPEC-implementation.md` 인용 기준:
  - Single human board: `:35`
  - §5.1 In Scope: `:60-71`
  - `approve_ceo_strategy` SPEC 목표: `:67` + `:768-772`
  - Issue Status: `:423-432`
  - atomic checkout contract: `:600-615`
- Hermes 표기:
  - `server/package.json:71` npm dependency
  - `server/src/adapters/registry.ts:128-138` 정적 import
  - `server/src/adapters/registry.ts:523` built-in 등록
  - upstream built-in / externalize 브랜치 plugin-only 분리 표기
- sandbox providers: 5종
  - `e2b`
  - `cloudflare`
  - `daytona`
  - `exe-dev`
  - `modal`
- hard cap budget enforcement:
  - `server/src/services/budgets.ts:213-258`
  - `cancelBudgetScopeWork`: `server/src/services/heartbeat.ts:9579-9604`

## 미해결·잔존 이견

0건.

## 사용자 지시·범위 제한

본 합의에 포함:

- `docs/report/00-overview.md`부터 `docs/report/10-research-map.md`까지의 본문 변경

제외:

- `docs/research/`
- SVG 그림
  - DEFER 별도 작업으로 처리

응답 파일 규칙:

- `docs/reviews/2026-05-24-chNN-codex-rN.md`

## 다음 단계

그룹 A 마지막 챕터:

1. `ch05` — `05-server-api.md`

`ch05`도 4 라운드 이내 합의한다.
