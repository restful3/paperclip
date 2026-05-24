# ch00 Round 4 — Claude Decisions (최종 라운드)

_2026-05-24, codex-a / 2 findings 분류 / MAX_ROUNDS 도달_

## ACCEPT — 모두 (2건)

### F1 — line 30: registry.ts 라인 범위 수정
- **AS-IS**: `server/src/adapters/registry.ts:511-524`
- **TO-BE**: `server/src/adapters/registry.ts:513-525`
- **근거**: 직접 sed 확인:
  - `:511` 빈줄, `:512` `function registerBuiltInAdapters() {`, `:513` `for (const adapter of [`
  - 잠깐 — 실제로 `:513` 가 `acpxLocalAdapter,` 첫 항목, `:525` 가 `httpAdapter,` 마지막 13번째 항목.
  - 본문 `:511-524` 는 적어도 1줄 부족하고 시작점도 어긋남. `:513-525` 가 13개 항목 영역 정확.

### F2 — line 73: SPEC.md future plugin 인용 라인 오프셋
- **AS-IS**: `KB는 future plugin으로 명시되어 있고(doc/SPEC.md:491, :500-502), 외부 매출/비용 회계도 future plugin이다(doc/SPEC.md:493, :517)`
- **TO-BE**: `KB는 future plugin으로 명시되어 있고(doc/SPEC.md:493, :500-502), 외부 매출/비용 회계도 future plugin이다(doc/SPEC.md:495, :519)`
- **근거**: sed 확인 — `:491` 은 "### Not V1" 헤더, `:493` 이 "- Knowledge base - a future plugin". `:495` 가 "- Revenue/expense tracking beyond token costs - a future plugin". `:517` 은 "Does not manage delivery infrastructure", `:519` 가 "Does not track external revenue/expenses".

## Codex 정합 판정 (Round 4 재검증)
- ✅ line 13 README.md:31 / line 30 ACPX 표기 / line 60 88/67 카운트 / line 60 execution-semantics 인용 — 모두 정합

## 변경 적용 파일
- `docs/report/00-overview.md` (line 30 + line 73, 2곳)

## MAX_ROUNDS 도달 — 챕터 종료 결정

Codex 가 매 라운드 새 정합성 이슈를 1\~2개씩 발견하는 패턴 (R1:4, R2:1, R3:1, R4:2). 정합성 검증은 무한 확장 가능하므로 **MAX_ROUNDS=4 의 cap 으로 종료**. R5 송부하지 않음.

남은 잠재 이슈는 ch04 와의 cross-chapter consistency (Codex R1 에서 지적한 ch04 adapter 카운트 STALE) — ch04 라운드에서 처리.
