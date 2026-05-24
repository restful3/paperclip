# ch08 Codex Handout — 2026-05-24

## 현재 작업 중인 문제

ch08 `docs/report/08-usage.md` Codex 정합성 합의 루프.

- 라운드 상태: R1-R5 완료, R6 송부 대기
- 리뷰 강도: 정합성 엄격 / 표현 중도 낮게 / 품질 가볍게
- 현재 목표: R5 board-claim 라인 정정까지 반영된 본문이 정합 상태인지 재확인

## 검토 대상 파일들

본문:

- `/media/restful3/data/workspace/paperclip/docs/report/08-usage.md`

주요 인용·검증 소스:

- `/media/restful3/data/workspace/paperclip/cli/src/index.ts`
- `/media/restful3/data/workspace/paperclip/cli/src/commands/run.ts`
- `/media/restful3/data/workspace/paperclip/cli/src/config/server-bind.ts`
- `/media/restful3/data/workspace/paperclip/cli/src/checks/deployment-auth-check.ts`
- `/media/restful3/data/workspace/paperclip/server/src/app.ts`
- `/media/restful3/data/workspace/paperclip/server/src/config.ts`
- `/media/restful3/data/workspace/paperclip/server/src/board-claim.ts`
- `/media/restful3/data/workspace/paperclip/server/src/routes/access.ts`
- `/media/restful3/data/workspace/paperclip/server/src/routes/agents.ts`
- `/media/restful3/data/workspace/paperclip/server/src/routes/approvals.ts`
- `/media/restful3/data/workspace/paperclip/server/src/routes/costs.ts`
- `/media/restful3/data/workspace/paperclip/server/src/services/companies.ts`
- `/media/restful3/data/workspace/paperclip/ui/src/App.tsx`
- `/media/restful3/data/workspace/paperclip/ui/src/api/client.ts`
- `/media/restful3/data/workspace/paperclip/ui/src/api/heartbeats.ts`
- `/media/restful3/data/workspace/paperclip/ui/src/components/ActiveAgentsPanel.tsx`
- `/media/restful3/data/workspace/paperclip/ui/src/components/AgentConfigForm.tsx`
- `/media/restful3/data/workspace/paperclip/ui/src/components/OnboardingWizard.tsx`
- `/media/restful3/data/workspace/paperclip/ui/src/components/transcript/useLiveRunTranscripts.ts`
- `/media/restful3/data/workspace/paperclip/ui/src/lib/new-agent-hire-payload.ts`
- `/media/restful3/data/workspace/paperclip/doc/DEPLOYMENT-MODES.md`
- `/media/restful3/data/workspace/paperclip/doc/DOCKER.md`
- `/media/restful3/data/workspace/paperclip/doc/SPEC-implementation.md`
- `/media/restful3/data/workspace/paperclip/docker/docker-compose.yml`
- `/media/restful3/data/workspace/paperclip/Dockerfile`
- `/media/restful3/data/workspace/paperclip/package.json`

## R1-R5 합의된 결정 사항

결정 파일:

- `/media/restful3/data/workspace/paperclip/docs/reviews/2026-05-24-ch08-decisions-r1.md`
- `/media/restful3/data/workspace/paperclip/docs/reviews/2026-05-24-ch08-decisions-r2.md`
- `/media/restful3/data/workspace/paperclip/docs/reviews/2026-05-24-ch08-decisions-r3.md`
- `/media/restful3/data/workspace/paperclip/docs/reviews/2026-05-24-ch08-decisions-r4.md`
- `/media/restful3/data/workspace/paperclip/docs/reviews/2026-05-24-ch08-decisions-r5.md`

리뷰 산출물:

- `/media/restful3/data/workspace/paperclip/docs/reviews/2026-05-24-ch08-codex-r1.md`
- `/media/restful3/data/workspace/paperclip/docs/reviews/2026-05-24-ch08-codex-r2.md`
- `/media/restful3/data/workspace/paperclip/docs/reviews/2026-05-24-ch08-codex-r3.md`
- `/media/restful3/data/workspace/paperclip/docs/reviews/2026-05-24-ch08-codex-r4.md`
- `/media/restful3/data/workspace/paperclip/docs/reviews/2026-05-24-ch08-codex-r5.md`

누적 finding: 15건 ACCEPT.

- R1: 8건 ACCEPT
  - `--bind lan` 주석에 "첫 실행(설정 없음) 때" 추가
  - 예산 API endpoint와 route-local 정의 분리
  - `NewAgent`/`AgentConfigForm` default env, adapter config, run policy 명시
  - CEO 자기 보고/전략 분해 자동 흐름을 현재 구현 흐름으로 정정
  - Dockerfile 라인 정정
  - board-claim 라인 정정 및 access route 추가
  - `/dashboard/live` 를 WebSocket 실시간이 아니라 live-runs API + run log polling으로 정정
- R2: 3건 ACCEPT
  - route-local 정의 + `/api` mount 분리
  - docker-compose `db`/`server` 라인 범위 보강
  - `/dashboard/live` 인용 보강
- R3: 2건 ACCEPT
  - "보드는 CEO 한 명만 직접 관리" 식 표현을 현재 UI/API의 전체 org tree 조회·생성·수정 가능성으로 정정
  - `/dashboard/live` server route와 UI API base client 인용 보강
- R4: 1건 ACCEPT
  - line 106 org tree 관리 범위 인용에 `server/src/routes/agents.ts:2540-2548`, `:677-685` 추가
- R5: 1건 ACCEPT 대기
  - line 192 board-claim 라인 정정: `server/src/routes/access.ts:2454-2495` → `server/src/routes/access.ts:2408-2453`

## 미해결·잔존 이견

없음.

- R5 까지 `0 REJECT`
- R5 까지 `0 DEFER`

## 사용자 지시·범위 제한

그룹 B 표준 handout:

- `/media/restful3/data/workspace/paperclip/docs/reviews/2026-05-24-grp-b-codex-handout.md`

범위:

- 포함: ch08 본문 `/media/restful3/data/workspace/paperclip/docs/report/08-usage.md`
- 제외: ch08 외 다른 챕터, `docs/research/`, SVG 그림
- ch10 합의 루프 DEFER 는 별도 commit `8a659b69` 으로 처리됨

적용해야 할 Group B 표준:

- DB `pgTable` 88개, `company_id` 보유 67개, schema 파일 79개
- built-in adapter 13개
- `packages/adapters` workspace adapter 10개, Hermes는 별도 외부 의존성
- 표기: `ACPX`, `grok-local` 포함
- `approve_ceo_strategy` 는 현재 구현 완료 흐름이 아니라 SPEC 목표(`doc/SPEC-implementation.md:67`, `:768-772`)
- hard cap 근거: `doc/SPEC.md:295-301`, `server/src/services/budgets.ts:213-258`, `:716-862`, `server/src/services/heartbeat.ts:9579-9604`

## 다음 단계

R6 재리뷰.

확인할 핵심:

- R5 finding인 board-claim 라인 정정이 본문에 반영되었는지:
  - 이전 잘못된 인용: `server/src/routes/access.ts:2454-2495`
  - 정정 인용: `server/src/routes/access.ts:2408-2453`
- R1-R5 누적 반영 후 `08-usage.md` 전체가 정합 상태인지 확인
- 남은 이슈가 없으면 정확히 `===CODEX_FINAL_APPROVAL===`
- 새 finding이 있으면 `/media/restful3/data/workspace/paperclip/docs/reviews/2026-05-24-ch08-codex-r6.md` 에 저장
