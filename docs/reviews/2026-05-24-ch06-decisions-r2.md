# ch06 Round 2 — Claude Decisions

_2026-05-24, codex-b / 5 findings 분류 (4 정합성 + 1 품질)_

## ACCEPT — 모두 (5건)

### F1 — line 19: Route 129 → 128
- **AS-IS**: `## 2. 라우팅 표 (Route 129개)`
- **TO-BE**: `## 2. 라우팅 표 (Route 128개)`
- **근거**: 검증 — `grep -E "<Route " App.tsx | wc -l = 128`. `<Routes>` 컨테이너 1개가 추가 매치되어 129가 잡혔던 것.

### F2 — line 94: prose `entity_type` 잔재 정정
- **AS-IS**: `들어온 LiveEvent 의 type/entity_type 으로 query key 를 invalidate`
- **TO-BE**: `들어온 LiveEvent 의 type/payload.entityType 으로 query key 를 invalidate`
- **근거**: Round 1 F11 에서 의사코드는 고쳤지만 본문 prose 는 누락. 일관성.

### F3 — line 77: 디렉터리 트리 `api/` 주석 완화
- **AS-IS**: `├── api/                 # fetch 래퍼 + zod 검증`
- **TO-BE**: `├── api/                 # fetch 래퍼 + 일부 응답 zod 검증`
- **근거**: Round 1 F10 의 본문 완화 (`api/` 가 packages/shared 의 타입을 공유, 일부 응답만 zod 검증) 와 일관. 트리 주석만 강한 표현 남아 있던 것.

### F4 — line 148: HMR "동일 origin" 정정
- **AS-IS**: `**HMR 도 동일 origin** — Vite WS 가 그대로 동작한다.`
- **TO-BE**: `**HMR** — Express 에 마운트된 Vite dev server 가 별도 HMR 포트(server/src/app.ts:381-392, 기본 serverPort+10000)로 WS 를 띄운다.`
- **근거**: `server/src/app.ts:381-392` `hmrPort = resolveViteHmrPort(opts.serverPort)`, `:92-97` 에서 기본 serverPort+10000. Codex 라인 인용 신뢰.

### F5 (품질) — line 117: WebSocket 의사코드 주석 중복
- **AS-IS**: `// …나머지 LIVE_EVENT_TYPES (heartbeat.run.queued/status, agent.status, activity.logged) 로 분기`
- **TO-BE**: `// 실제 구현은 heartbeat.run.queued/status, agent.status, activity.logged 등을 별도 handler 로 처리`
- **근거**: 바로 위 `if` 가 이미 `activity.logged` 처리. "나머지" 라는 표현이 의사코드 흐름과 모순.

## REJECT / DEFER
- 없음.

## 변경 적용 파일

- `docs/report/06-ui-and-board.md` (총 5곳 수정)

## Round 3 송부 의도

Round 1 의 후속 정합 작업 + 1 finding 모두 해결. `===CODEX_FINAL_APPROVAL===` 기대.
