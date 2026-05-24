# ch07 Round 2 — Claude Decisions

_2026-05-24, codex-b / 4 findings (3 정합성 + 1 품질)_

## ACCEPT (3건)

### F1 — line 63: §4-pause-behavior → SPEC.md:231-240 인용
- **근거**: `§4-pause-behavior` 는 본문에만 존재하는 가짜 앵커. 실제 정의는 `doc/SPEC.md:231-240` "Pause Behavior".

### F2 — line 140: README 직접 인용 정정
- **AS-IS**: `*"recurring jobs you have to remember to manually kick off"*라는 README 문제 정의`
- **TO-BE**: `README 의 recurring jobs 문제 정의(README.md:139)의 직접적 해법이다.` (단순화 옵션 — 따옴표 인용 대신 라인 인용)
- **근거**: 원문은 `You have recurring jobs (customer support, social, reports) and have to remember to manually kick them off.` — 본문 인용이 축약되어 직접 인용으로는 부정확.

### F4 (품질) — line 63: 표 1 정렬 주장 제거
- **AS-IS**: "*영향 반경*으로 정렬되어 있다 — 위에서 아래로 갈수록 ... 좁혀진다. ... 위에서 아래까지 모든 행이"
- **TO-BE**: "실행 중단, 작업 통제, 예산, 결정, 산출물 동결까지 보드가 가진 hard 권한을 한데 모은 것이다. ... 표의 모든 행이"
- **근거**: 표 실제 순서 (에이전트 일시정지 → 회사 일시정지) 가 "위→아래 좁혀진다" 와 충돌. 정렬 주장 빼는 게 단순.

## DEFER (1건)

### F3 — `docs/research/10-llm-cost-budgeting.md:39` 라인 정합 (research scope 밖)
- **이유**: 사용자 명시 — "범위: docs/report/00\~10 (research/ 제외)". ch07 본문 자체는 이미 정합.
- **처리**: 합의 루프 후 별도 작업 — `docs/research/10-llm-cost-budgeting.md` 의 costs.byProject `:454~492` → `:454-502` 업데이트.

## 변경 적용 파일

- `docs/report/07-governance-cost.md` (3곳)

## Round 3 송부 의도

본문 정합성 + 표현 거의 마무리. FINAL_APPROVAL 기대.
