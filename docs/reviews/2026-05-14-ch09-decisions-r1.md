# Ch09 — 라운드 1 결정

## ACCEPT (6건)

### [9-1] §2 표 1 #1 어댑터 계약 두 층 혼합 ✅ ACCEPT
- HANDOFF.md §3 합의: 장기 SPEC `invoke/status/cancel`(`doc/SPEC.md:207-215`) vs 현재 구현 `execute`+`testEnvironment` 필수(`packages/adapter-utils/src/types.ts:349-352`) — 두 층.
- 본문 셀이 두 층을 한 줄로 섞어 놓음.
- **결정**: 셀을 "장기 SPEC … vs 현재 구현 …" 두 층 표현으로 정정.

### [9-2] §2 표 1 #3 atomic checkout 라인 범위 ✅ ACCEPT
- HANDOFF.md §3 합의: `server/src/services/issues.ts:3593-3736`.
- 본문 `:3409\~3525` 는 일반 issue update 경로 (Codex 검증).
- **결정**: `:3593-3736` 으로 정정.

### [9-3] §2 표 1 #4 보수적 회복 인용 보강 ✅ ACCEPT (핸드오프 §5 예고 항목)
- `doc/execution-semantics.md:407-422` §12 "What This Does Not Mean" — preserve ownership / retry once / explicit recovery action / escalate visibly 4문구 직접 명시.
- `doc/SPEC-implementation.md:434-442` V1 non-terminal liveness rule.
- **결정**: 셀 인용을 `doc/execution-semantics.md:407-422` + `doc/SPEC-implementation.md:434-442` 로 교체하고, 4문구 정확히 인용.

### [9-4] §2 표 1 #6 하드 일시정지 근거 분해 ✅ ACCEPT
- `server/src/services/agent-start-lock.ts:32` 의 `withAgentStartLock` — queued run start 직렬화.
- `server/src/services/heartbeat.ts:8610-8618` `budgets.getInvocationBlock` — 직접 실행 차단.
- `server/src/services/heartbeat.ts:9431-9456` `cancelBudgetScopeWork` — paused scope 작업 취소.
- `doc/SPEC.md:295-300` hard ceiling 정책.
- **결정**: `agent_start_lock 서비스` 단독 근거를 세 메커니즘 조합으로 풀어 적음.

### [9-5] §3 표 2 #3 "단일 보드" 과소서술 ✅ ACCEPT
- `packages/db/src/schema/company_memberships.ts:4-13` — 회사별 user principal + role.
- `packages/shared/src/constants.ts:624-630` — human 4 role: owner/admin/operator/viewer.
- `packages/db/src/schema/instance_user_roles.ts` — `instance_admin` 단일 종류, 그러나 multi user 가능.
- `doc/SPEC.md:489-493` — multi-member boards · advanced governance = Not V1.
- `server/src/__tests__/issue-execution-policy.test.ts:98-105` — `approvalsNeeded > 1` 거부.
- **결정**: 약점 명칭을 "단일 보드" → "고급 거버넌스 부재 (V1)" 로 좁히고, 4-eye 결재·다중 승인 정책·multi-member boards 가 Not V1 임을 본문에 명시.

### [9-9] §2 표 1 #10 `~` 이스케이프 누락 ✅ ACCEPT
- 본문 line 35: `~/.paperclip/adapter-plugins.json` — `~` 이스케이프 안 됨.
- HANDOFF §4 마크다운 관습: 본문 `~` 는 `\~` 이스케이프.
- **결정**: `\~/.paperclip/adapter-plugins.json` 로 수정.

## REJECT — 없음

## DEFER
- 9-6 / 9-7 / 9-8 은 Codex 재검증 OK 표시 (수정 없음, 그대로 보존)
- 라인 범위 ±1\~2 nit / 영문 용어 취향 / 캡션 문체

## 다음
본문 수정 → `2026-05-14-ch09-changes-r1.md` → r2 재리뷰 요청.
