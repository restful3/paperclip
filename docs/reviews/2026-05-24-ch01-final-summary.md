# ch01 Final Summary

_2026-05-24, codex-a / 3 rounds / `===CODEX_FINAL_APPROVAL===` 도달_

## Outcome: **approved** (3 rounds, FINAL_APPROVAL)

## 변경 통계

| 라운드 | finding | ACCEPT | DEFER | LOC |
|---|---|---|---|---|
| R1 | 7 | 6 | 1 (SVG) | ~12 |
| R2 | 2 (품질) | 2 | 0 | 2 |
| R3 | FINAL_APPROVAL | - | - | 0 |
| **합** | **9** | **8** | **1** | **~14** |

## 본문 변경

`docs/report/01-architecture.md`:
- line 25-27, 31: 9개 → 10개 adapter workspace + 인용 라인 정정 (server :47-56 71 / ui :37-46 55 / cli :40-49)
- line 28, 117: 80+ → 88개 테이블 (ch00 정합)
- line 39: registry.ts `:511-524` → `:513-525`, server/package.json `:47-55` → `:47-56`, hermes 표현 분리 (import vs 등록)
- line 58: detect-port `:64` → `:65`
- line 97: `-r typecheck`/`-r build` → `typecheck`/`build`
- line 99: §6 제목 "두 종류의 인스턴스 디렉터리" → "인스턴스 디렉터리 레이아웃"

## DEFER (별도 작업)

| 항목 | 처리 시점 |
|---|---|
| §1 그림 1-1 SVG 결락 (`assets/fig-02-monorepo.svg`) | 합의 루프 후 별도 — cursor-cloud, grok-local 추가, Hermes 분리 표기 |

## Context 사용

- codex-a Context 0% → 81% (ch00 4 rounds + ch01 3 rounds 후)
- ch02 시작 전 **handout-then-clear 예정**
