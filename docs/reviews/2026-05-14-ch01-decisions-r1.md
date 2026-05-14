# 챕터 01 — 라운드 1 결정 로그

## ACCEPT (7)

### A1. adapter 카운트 11 → 12 (`:39`) [기준 1]
- 검증: `server/src/adapters/registry.ts:480-493` 12개. `server/package.json:47-55` 9개 워크스페이스 어댑터 + `:70` `hermes-paperclip-adapter` npm + process/http 내장 = 12.
- 적용: 본문 11→12, cursor-cloud 신규 포함, 패키지형 9개 + hermes(npm) + generic 2 구분 명시.

### A2. 어댑터 표준 트리 주석: invoke/status/cancel → execute/testEnvironment (`:47`) [기준 1]
- Ch00 합의 사항. 코드 1 주석 + 본문 한 줄 정정.

### A3. 표 1 의존 관계 부정확 (`:25-33`) [기준 1]
- 검증: 
  - `server` 의존 `mcp-server` 아님 (`server/package.json:45-79`)
  - `adapter-utils` 는 `shared` 의존 아님 (`packages/adapter-utils/package.json:42-45` dev-only)
  - `plugins` 는 단일 패키지가 아님 — `packages/plugins/sdk` 만 있고 host runtime은 `server/src/services/plugin-*`
- 적용: 표 1 의존 열 다시 작성, `packages/plugins` 행을 `packages/plugins/sdk` 로 정정 + plugin host runtime 위치 보강.

### A4. mcp-server DB 의존 표현 부정확 (`:30`) [기준 1]
- 검증: `packages/mcp-server/package.json:45-49` deps = sdk + shared + zod. DB 의존 없음. 동작은 REST API 호출.
- 적용: "shared · DB(REST 경유)" → "shared · REST API (fetch)".

### A5. dev 포트 3100 고정 표현 (`:58`, `:63`) [기준 1]
- 검증: `server/package.json:64` `"detect-port": "^2.1.0"` 의존. 3100이 사용 중이면 다음 free port.
- 적용: "같은 포트(`3100`)에서 서빙" → "기본 요청 포트 3100 — 점유 중이면 detect-port가 다음 사용 가능한 포트로 올린다". 코드 2 주석 동시 수정.

### A6. 인스턴스 디렉터리 코드 3 축약 (`:101-113`) [기준 1, 좁힘]
- 적용: 코드 3 직전에 "핵심 경로만 추리면" 한 줄 추가. 전체 레이아웃 인용은 `doc/DEVELOPING.md` 링크로 위임 (모든 경로 옮기면 표 비대화).

### A7. fork 포트 동작 표현 (`:115`) [기준 1]
- 검증: 일반 동작은 fork명 무관. detect-port 기반.
- 적용: "`HenkDz/paperclip` 같은 포크는 포트 3101+ 로 자동 자리잡는다" → "`PAPERCLIP_HOME` / `PAPERCLIP_INSTANCE_ID` 로 인스턴스를 분리하고, 포트는 detect-port가 사용 가능한 값으로 조정한다".

### A8. line citation 보강 [기준 5, 좁힘]
- 수치/패키지/명령 같은 검증 가능한 주장 단락에 path:line 인라인 인용:
  - 워크스페이스 선언: `pnpm-workspace.yaml`
  - server 의존: `server/package.json:47-79`
  - mcp-server: `packages/mcp-server/package.json:45-49`
  - adapter-utils: `packages/adapter-utils/package.json:42-45`
  - plugin-sdk: `packages/plugins/sdk/package.json:110-113`
  - dev 포트: `server/package.json:64`
  - 어댑터 12 + registry: `server/src/adapters/registry.ts:480-493`

## DEFER (2) — Codex 결정 수용

- D1. "의존이 위에서 아래로만 흐른다" — 표 1 정정 후 재판단
- D2. "70+ 페이지" — 정의 모호, 라우트 근거 보강은 차후

## OUT-OF-SCOPE — 그대로 수용

- `assets/fig-02-monorepo.svg` 내부 라벨 — 별도 그림 작업
- `_pdf-build` 산출물 — 본문 합의 후 재빌드

## 누적 LOC

- Ch00: ~32 LOC + Ch01 r1: ~25 LOC 예상 = 누적 ~57 / 200 checkpoint
