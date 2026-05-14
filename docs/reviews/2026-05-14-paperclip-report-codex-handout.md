# Paperclip 학습 리포트 검토 handout

## 현재 작업 중인 문제

Paperclip 학습 리포트 11챕터(`docs/report/00~10`)를 5개 기준(사실 정확성 / 표현 효율성 / 한국어 자연성 / 마크다운 관습 / 인용 완결성)으로 라운드별 합의 검토하는 작업이다.

## 검토 대상 진행 상황

- Ch00 ✅ 합의 완료 (r3, FINAL_APPROVAL)
- Ch01 ✅ 합의 완료 (r3, FINAL_APPROVAL)
- Ch02 ✅ 합의 완료 (r2, FINAL_APPROVAL)
- Ch03 — 라운드 4 대기 (r3에서 1줄 cancelled 표현 정정 적용됨)
- Ch04~10 — 대기

## 합의된 결정 사항

- built-in adapter 12개 (런타임 10 + generic 2). `cursor_cloud` 포함. 패키지형 9개 + `hermes_local`(npm) + `process`/`http`(generic). 출처 `server/src/adapters/registry.ts:480-493`.
- 어댑터 계약: 장기 SPEC `invoke/status/cancel`(`doc/SPEC.md:207-215`) vs 현재 구현 `execute/testEnvironment`(`packages/adapter-utils/src/types.ts:349-352`) 두 층 분리.
- issue status 7개: backlog/todo/in_progress/in_review/done/blocked/cancelled. `open` 없음. terminal은 done/cancelled.
- issue `goal_id`/`parent_id` nullable + fallback service.
- issues는 `checkout_run_id`/`execution_run_id` 로 heartbeat_runs 직접 참조 (activity_log.run_id 는 보조 다리).
- cost_events: `agent_id` 필수, 나머지 4 FK(issue/project/goal/heartbeat_run) nullable.
- pgTable 86개 정의 / 78개 스키마 파일.
- 마이그레이션 위치 `packages/db/src/migrations/`(`drizzle.config.ts:5`), 자동 적용 조건은 dev/server boot 경로에 있음.
- 플러그인 테이블 실명: `plugins`, `plugin_company_settings`, `plugin_database_namespaces`, `plugin_migrations`, `plugin_state`.
- V1 Out 항목 분류: KB·매출 회계는 future plugin(`doc/SPEC.md:491,500-502,493,517`), ClipHub·multi-board governance·self-healing 은 V1 밖 거버넌스 확장.
- README 문제 6개 / 페르소나 7개.
- PRODUCT.md 5 원칙(`doc/PRODUCT.md:61-71`) ≠ SPEC §13 9 원칙(`doc/SPEC.md:521-531`); 리포트 표 1의 5개는 두 문서 합쳐 학습용 추림.
- Startup reconciliation 7-pass (reap → promote scheduled → resume queued → stranded → liveness → silent → productivity reviews) (`server/src/index.ts:676-780`).
- source-scoped recovery action: `issue_recovery_actions` 테이블 + service. 활성(`active`/`escalated`) recovery action은 source issue당 1개, 같은 source 내 `(cause, fingerprint)` 중복 방지(`packages/db/src/schema/issue_recovery_actions.ts:61-66`).
- silent active-run watchdog 은 recovery action 으로 표면화, issue-backed 평가는 `stale_active_run_evaluation` 이슈.

## 마크다운 관습

- CJK bold 공백: bold 내부 **구두점 포함된 경우에만** 닫는 `**` 뒤 공백. 단순 `**Paperclip**은`은 그대로.
- 캡션 `**그림 N. 〈제목〉**` / `**표 N. 〈제목〉**`. 본문 참조는 평문.
- 외부 인용 그림은 메인 번호로 통합 (`1-A` 류 폐기). 단, 챕터 간 글로벌 번호 정합화는 final-summary 단계에서 일괄.
- 코드 펜스 언어 식별자 필수. 본문 `~`는 `\~`.

## 인용 정책

수치·리스트·계약·원칙·테이블 주장에 `path:line` 또는 URL. 만연 인용은 피함.

## 종결 신호

라운드 N에서 ACCEPT 0이면 응답 첫 줄 정확히 `===CODEX_FINAL_APPROVAL===`.

## 미해결 / 잔존 이견

0건. Ch03 r4 가 final approval 받기를 기대.

## 사용자 지시 / 범위 제한

11챕터 다라운드 자동 합의 루프. 전부 끝나면 일괄 보고. Paperclip 코드 변경 0건. 산출물은 `docs/report/`(본문) + `docs/reviews/`(검토 산출물).

## 참고 파일

Ch03 라운드 1~3 기록:

- `docs/reviews/2026-05-14-ch03-decisions-r1.md`
- `docs/reviews/2026-05-14-ch03-changes-r1.md`
- `docs/reviews/2026-05-14-ch03-codex-r1.md`, `r2.md`, `r3.md`

## 다음 단계

Ch03 라운드 4 — 본문 `:65-68` cancelled 설명 1줄 수정만 확인. 합의되면 final approval. 이후 Ch04 (`docs/report/04-adapters-and-skills.md`, CHANGE-IMPACT 상 STALE 등급) 시작.
