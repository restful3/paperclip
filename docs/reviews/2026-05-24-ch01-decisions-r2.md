# ch01 Round 2 — Claude Decisions

_2026-05-24, codex-a / 2 findings 분류 (모두 품질)_

## ACCEPT (2건)

### F1 — line 99: §6 제목 정정
- **AS-IS**: `## 6. 두 종류의 인스턴스 디렉터리`
- **TO-BE**: `## 6. 인스턴스 디렉터리 레이아웃`
- **근거**: 본문은 단일 instance root 의 런타임 데이터 레이아웃 설명. "두 종류" 라는 분류 없음.

### F2 — line 97: PR 전 검증 표기 일관성
- **AS-IS**: `\`test\`/\`test:e2e\`/\`-r typecheck\`/\`-r build\` 가 PR 전 검증 라인이다`
- **TO-BE**: `\`test\`/\`test:e2e\`/\`typecheck\`/\`build\` 가 PR 전 검증 라인이다`
- **근거**: 위 표 2 가 `pnpm typecheck`, `pnpm build` 표준 명령으로 표기. 일관성.

## Codex 정합 판정 (Round 2 재검증)
- ✅ adapter workspace 10개 + package.json 인용 라인
- ✅ registry.ts:513-525, :128-138 import, :523 등록
- ✅ detect-port :65
- ✅ 88개 테이블 표기 (ch00 정합)
- DEFER 된 SVG 결락 재지적 없음 — 약속대로 별도 작업

## 변경 적용 파일

- `docs/report/01-architecture.md` (line 97, 99 — 2곳)

## Round 3 송부 의도

단순 품질 정정만 — `===CODEX_FINAL_APPROVAL===` 기대.
