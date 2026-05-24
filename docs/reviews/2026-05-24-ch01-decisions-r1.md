# ch01 Round 1 — Claude Decisions

_2026-05-24, codex-a / 7 findings 분류 (6 정합성 + 1 품질)_

## ACCEPT (6건)

### F1 — line 25-27/31: 9개 → 10개 adapter workspace + 인용 라인 정정
- **AS-IS**: 본문 4곳 "9개", 인용 라인이 server/package.json:47-70, ui:37-54, cli:40-57
- **TO-BE**: "10개" (4곳), server/package.json:47-56, 71 / ui:37-46, 55 / cli:40-49
- **검증**: `ls packages/adapters/` = 10개. server/package.json:47-56 가 10개 adapter workspace + :71 hermes. ui:37-46 + :55 hermes. cli:40-49 (hermes 없음).

### F2 — line 39: registry.ts:511-524 → :513-525
- **검증**: ch00 R4 와 동일 정정. `:513` 첫 항목, `:525` 13번째 (httpAdapter).

### F3 — line 39: server/package.json:47-55 → :47-56
- **검증**: :47-55 는 9 라인, :56 의 pi-local 빠뜨림. :47-56 가 10개 전체.

### F4 — line 39: hermes 표현 변경
- **AS-IS**: "hermes_local 은 hermes-paperclip-adapter npm 의존성을 registry.ts:128-138 에서 정적으로 등록한다"
- **TO-BE**: "hermes_local 은 hermes-paperclip-adapter 를 정적으로 import 하고(registry.ts:128-138), built-in 등록 배열에 포함된다(registry.ts:523)"
- **검증**: `:128-138` 가 import 라인 (sed 확인), `:523` 가 hermesLocalAdapter 배열 포함.

### F6 — line 58: detect-port :64 → :65
- **검증**: server/package.json:65 = "detect-port", :64 = "chokidar".

### F7 — line 28, 117: 80+ → 88개 테이블 (ch00 정합성)
- **검증**: ch00 최종 본문 "88개 테이블 중 67개" 와 일관.

## DEFER (1건)

### F5 — §1 그림 1-1 SVG 수정
- **이유**: SVG 직접 편집 필요 (cursor-cloud, grok-local 추가, Hermes 분리 표기). 텍스트 수정 도구로 어려움.
- **처리**: 합의 루프 완료 후 별도 작업 — `docs/report/assets/fig-02-monorepo.svg` 갱신 + PDF 재빌드.

## 변경 적용 파일

- `docs/report/01-architecture.md` (총 9 Edit 적용)
- DEFER: `docs/report/assets/fig-02-monorepo.svg`

## Round 2 송부 의도

본문 정합성 1차 정리 완료. SVG 결락은 별도 후속 작업으로 분리 — 합의는 본문 기준.
