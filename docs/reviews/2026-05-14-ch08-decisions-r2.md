# Ch08 — 라운드 2 결정

## ACCEPT (4건)

### [8-15] §4 월 예산 위치 ✅ ACCEPT
- `ui/src/pages/CompanySettings.tsx:245-374` 확인 — General(name/description) + Appearance(logo/brand color/attachment size limit) 만, 월 예산 입력 없음.
- `server/src/routes/costs.ts:280` 의 `PATCH /companies/:companyId/budgets` 가 실제 budget 변경 엔드포인트.
- **결정**: r1 패치의 회사 설정 주석에서 `월 예산(budget_monthly_cents)` 항목을 분리, "비용/예산 API" 경로로 옮김.

### [8-16] §8 단독 docker run `BETTER_AUTH_SECRET` 누락 ✅ ACCEPT
- `Dockerfile:76` 기본값 `PAPERCLIP_DEPLOYMENT_MODE=authenticated`.
- `doc/DOCKER.md:31-37` one-liner 도 `-e BETTER_AUTH_SECRET=$(openssl rand -hex 32)` 포함.
- **결정**: 단독 `docker run` 예시에 `-e BETTER_AUTH_SECRET=$(openssl rand -hex 32)` 추가 + Dockerfile 기본값 인용 1줄.

### [8-17] §9 `PAPERCLIP_DEPLOYMENT_MODE` / `EXPOSURE` 누락 ✅ ACCEPT
- `server/src/config.ts:165` — `deploymentMode` 기본은 `local_trusted`.
- `:177-180` — `local_trusted` 면 `deploymentExposure` 가 자동 private 강제.
- env-only 로 `authenticated/public` 만들려면 두 변수 모두 명시 필요.
- **결정**: 코드 6 블록에 두 변수 추가 + 도입 단락에 기본값 fallback 노트.

### [8-18] §9 board-claim 흐름 일반화 과장 ✅ ACCEPT
- `doc/DEPLOYMENT-MODES.md:115-126` §7 — "Local Trusted -> Authenticated Claim Flow", 조건: 모드 = authenticated **AND** 유일한 instance admin 이 `local-board`.
- `server/src/board-claim.ts:47-61` 코드도 동일 조건.
- **결정**: "보드 첫 사용자가 board-claim URL 로 승격" → "local_trusted → authenticated 전환 시 admin 단일 local-board 조건일 때만" 으로 정정.

## REJECT — 없음

## DEFER

- 라인 범위 nit, 영문 용어 취향, 코드 캡션 문체 (계속 DEFER)

## 변경 위치 요약

| 항목 | 패치 위치 |
|---|---|
| 8-15 | §4 첫 화면 끝 단락의 `/company/settings` 주석 |
| 8-16 | §8 코드 5 단독 `docker run` 라인 + 도입 단락 후미 |
| 8-17 | §9 코드 6 "authenticated/public 모드 필수" 블록 + 본문 |
| 8-18 | §9 코드 6 직후 board-claim 단락 |
