# ch04 Round 2 — Claude Decisions

_2026-05-24, codex-a / 2 정합성 findings (모두 ACCEPT)_

## ACCEPT (2건)

### F1 — line 23: research note 범위 명확화
- **변경**: "(자세한 내용 + 인용은 docs/research/04-...)" → "주요 headless CLI 계열(Codex, Cursor CLI, Gemini, OpenCode, Pi)의 자세한 비교와 인용은 docs/research/04-... 에서 다룬다"
- **근거**: research note 는 5개만 다룸. Claude/Cursor Cloud/Grok/ACPX/OpenClaw/Hermes 까지 포함하는 표 1 전체 근거로 적기엔 과함.

### F2 — line 95-105: 코드 2 plugin install vs adapter install 구분
- **변경**: 본문 + 코드 블록 재작성
  - `pnpm paperclipai plugin install` → `POST /api/adapters/install` 또는 Board > Adapter Manager
  - 인용 추가: `routes/adapters.ts:222-326`, `services/adapter-plugin-store.ts:2-50`
- **근거**: `paperclipai plugin install` 은 일반 plugin (`/api/plugins/install`). 외부 adapter 설치는 별도 `/api/adapters/install` → `registerWithSessionManagement()` + `addAdapterPlugin()`. 인증 확실.

## 변경 적용 파일
- `docs/report/04-adapters-and-skills.md` (2곳, F2 는 큰 변경)

## Round 3 송부 의도
FINAL_APPROVAL 기대.
