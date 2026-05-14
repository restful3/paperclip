# 챕터 00 — 라운드 2 결정 로그

## ACCEPT (5)

### A1. 표 1 #4 셀: 원칙 + 구현 분리
- `00-overview.md:57`
- 표 셀 슬로건만으로는 후속 단락 정정과 시각적으로 충돌. 셀 안에서 "원칙: 모든 issue가 goal로 도달 / 구현: nullable + fallback" 두 줄로.
- 근거: `packages/db/src/schema/issues.ts:28-29`, `server/src/services/issue-goal-fallback.ts:3-12`

### A2. SPEC.md 인용 분리
- `00-overview.md:73`
- 검증: `:491` KB future plugin, `:500-502` KB anti-goal, `:493`/`:517` 매출 회계 future plugin. ClipHub는 future plugin이 아니라 `doc/SPEC-implementation.md:80` V1 Out 항목.
- 셋을 분리해 인용.

### A3. 좁힌 CJK bold 잔여 위반 3곳
- `:13` `**에이전트를 고용·조직·예산 통제·감사하는 회사 운영체제**다` (안에 `·`)
- `:30` (이전 `:29`였으나 라운드 1 이후 한 줄 늘어남) `**그림 1. Paperclip의 3-zone 큰 그림**은` (안에 `.`)
- `:73` `**KB · 매출 회계 · 공개 마켓플레이스는 의도적으로 코어 밖**이다` (안에 `·`)
- 조치: 닫는 `**` 뒤 공백 삽입 (CLAUDE.md 1순위 해법).

### A4. 그림 1-A → 그림 2 통합
- `00-overview.md:40-42`
- `**그림 N. 〈제목〉**` 규칙에 `1-A` 형식이 맞지 않음. 외부 인용 보조 번호 체계는 일관성 없이 운영하기 어려움. 챕터 00의 외부 인용 그림은 1개뿐이므로 메인 번호로 통합.

### A5. line citation 정밀화
- `:60` `issue-goal-fallback.ts` → `:3-12`
- `:60` `doc/execution-semantics.md`의 8\~12장 회복 규칙 → 핵심 줄 인용 (Codex 추천 `:298-305`, `:369-420`은 추후 챕터 03에서 정밀 검증; 챕터 00에서는 8\~12장 인용은 그대로 두되 "회복 규칙" 단락 끝에 line range 보강 시도)

## DEFER (2) — Codex 결정 수용

- D1. orchestrator-workers 비유 톤 — 전체 리포트 톤 합의 후 일괄 결정
- D2. "80여 개 / 약 60개" 테이블 수치 — 자동 산출 어려움, 별도 검증 도구 도입까지 보류

## OUT-OF-SCOPE — Codex 결정 수용

- LOC checkpoint 운영은 챕터 00 범위 밖
- 챕터 03 SPEC/구현 어댑터 계약 — 챕터 03 라운드에서 따로
