# ch00 Round 3 — Claude Decisions

_2026-05-24, codex-a / 1 finding 분류_

## ACCEPT

### F (Round 3) — line 30: ACP-x → ACPX 표기 통일
- **AS-IS**: `..., OpenCode, Pi, ACP-x, OpenClaw 게이트웨이, Hermes`
- **TO-BE**: `..., OpenCode, Pi, ACPX, OpenClaw 게이트웨이, Hermes`
- **근거**: 직접 검증:
  - `server/src/adapters/registry.ts:269`: `type: "acpx_local"`
  - `packages/adapters/acpx-local/src/index.ts:3-4`: `type = "acpx_local"`, `label = "ACPX (local)"`
  - ch04 표 1 도 `acpx-local` 로 표기. ch00 만 `ACP-x` (하이픈) 로 다름 — 일관성.

## Codex 정합 판정 (Round 3 재검증)

- ✅ line 13 슬로건 README.md:31 인용
- ✅ line 30 built-in adapter 13개 + registry.ts:511-524
- ✅ line 60 카운트 88/67
- ✅ line 60 execution-semantics 인용 :155-172, :174-197, :207-240

## 변경 적용 파일

- `docs/report/00-overview.md` (line 30, 1곳 수정)

## Round 4 송부 의도

이번이 마지막 변경 (MAX_ROUNDS=4 의 4번째 라운드). 표기 수정 1건만 — `===CODEX_FINAL_APPROVAL===` 기대.
