# Ch10 — 라운드 1 결정

## ACCEPT (4건)

### [10-1] R9 "recovery 4-pass" → 7-pass ✅ ACCEPT
- HANDOFF.md §3 합의: startup reconciliation 7-pass(`server/src/index.ts:676-780`).
- **결정**: R9 셀에서 "recovery 4-pass" → "startup/periodic reconciliation 7-pass" 정정.

### [10-2] R7 "8개 패키지" → 실제 24개 ✅ ACCEPT
- `pnpm-workspace.yaml` globs: `packages/*`, `packages/adapters/*`, `packages/plugins/*`, `packages/plugins/examples/*`, `server`, `ui`, `cli` (sandbox-providers / 일부 example 제외).
- `pnpm -r list --depth -1 --parseable | wc -l` = 25 (루트 포함). 실제 workspace 패키지 24.
- `packages/adapters/` 디렉터리 = 9개 어댑터 패키지.
- **결정**: "8개 패키지" → "24개 워크스페이스 패키지(adapters 9, plugins, server, ui, cli, …)" 정정.

### [10-3] §4 "3-메서드 + 4개 진입점" 두 층 분리 ✅ ACCEPT
- 장기 SPEC: `invoke/status/cancel` 3-method (`doc/SPEC.md:207-215`).
- 현재 구현: `ServerAdapterModule` 의 `execute` + `testEnvironment` 2 필수 (`packages/adapter-utils/src/types.ts:349-431`).
- 4 진입점: adapter package `exports` 의 `.`, `./server`, `./ui`, `./cli` (`packages/adapters/claude-local/package.json:15-20`).
- **결정**: 두 층 + 진입점 의미 분리해서 본문 1문장 재작성.

### [10-4] 그림 본문 도입 굵게 표시 위반 ✅ ACCEPT
- HANDOFF.md §4: 캡션 `**그림 N. ...**`, 본문 그림 참조는 평문.
- 위반 라인: line 11 (`**그림 10**`), line 46/52/58/64 (`**그림 10-N. 제목**`).
- 캡션 라인 line 13/48/54/60/66 은 그대로 유지.
- **결정**: 위반 5곳을 평문 `그림 10` / `그림 10-N (제목)` 형태로 수정.

## REJECT — 없음

## DEFER
- 10-5 budgets.ts (31 KB) · budget_incidents 재검증 OK
- 외부 캡션 문체 · 라인 범위 nit
