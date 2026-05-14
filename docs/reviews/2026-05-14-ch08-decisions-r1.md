# Ch08 — 라운드 1 결정 (ACCEPT / REJECT / DEFER)

## ACCEPT (6건)

### [8-4] §4 회사 생성 입력 — 첫 화면 과장 ✅ ACCEPT
- Codex 인용 검증: `ui/src/components/OnboardingWizard.tsx:681,700` — step 1 화면은 `Company name` + `Mission / goal (optional)` 두 입력만.
- `handleStep1Next` (`:392`)는 `companiesApi.create({ name })` 만 호출, `issuePrefix` 는 서버 응답에서 받음.
- `createCompanySchema` (`packages/shared/src/validators/company.ts:16-21`) 는 `name`/`description`/`budgetMonthlyCents`/`attachmentMaxBytes` 만 받고, `brandColor` 는 `updateCompanySchema` 에만 있음.
- `createCompanyWithUniquePrefix` (`server/src/services/companies.ts:140-149`) 가 회사명에서 prefix 파생.
- **결정**: §4 (A) 항목을 첫 화면 실제 입력 2개로 정정, 부가 항목은 회사 설정에서 채운다는 주석.

### [8-6] §5 `/agents/new` 입력 필드 표 — 실제 화면과 다름 ✅ ACCEPT
- `ui/src/pages/NewAgent.tsx:66-71` 의 useState: `name`, `title`, `role`, `reportsTo`, `configValues`, `selectedSkillKeys` — `icon`, `capabilities`, `budgetMonthly` 없음.
- `ui/src/lib/new-agent-hire-payload.ts:38` 에서 `budgetMonthlyCents: 0` 으로 고정.
- `capabilities` 는 `AgentConfigForm.tsx` 의 edit-only Identity 섹션.
- 첫 에이전트는 자동 `role=ceo`, `name=CEO`, `title=CEO` (`:107-110`).
- **결정**: 표에서 `icon` / `capabilities` / `budget_monthly_cents` 행 제거하고, 노트로 "생성 후 에이전트 설정에서 채움" 명시.

### [8-8] §5 본문 그림 참조 마크다운 위반 ✅ ACCEPT
- 본문 `**그림 8**은 …` (line 99) — HANDOFF.md §4 / 핸드오프 마크다운 관습 "본문 그림 참조는 평문" 위반.
- 캡션 자체 (`**그림 8. …**`, line 101) 는 규칙 준수.
- **결정**: `**그림 8**은` → `그림 8은` 으로 수정.

### [8-11] §8 Docker compose 설명 — 실제 구성 불일치 ✅ ACCEPT
- `docker/docker-compose.yml` — `db` (postgres:17-alpine) + `server` (Paperclip Dockerfile) **두 서비스 모두 띄움**.
- `doc/DOCKER.md:84-89` — full stack 호출은 `BETTER_AUTH_SECRET=$(openssl rand -hex 32) docker compose -f docker/docker-compose.yml up --build`.
- **결정**: 코드 5 와 도입 단락을 풀스택 compose 로 수정.

### [8-12] §9 호스티드 — env 3개로 불완전 ✅ ACCEPT
- `cli/src/checks/deployment-auth-check.ts:28-39` — `authenticated` 모드에서 `BETTER_AUTH_SECRET` 필수.
- `:51-60` — `authenticated/public` 에서 `auth.publicBaseUrl` (또는 `PAPERCLIP_PUBLIC_URL`) 필수.
- `docker/docker-compose.yml:30-31` 도 둘 다 명시.
- **결정**: 코드 6 에 `BETTER_AUTH_SECRET` + `PAPERCLIP_PUBLIC_URL` 추가, 도입 단락에 "3개"가 아닌 "핵심 환경 변수" 로 수정 + auth 변수 1줄 추가.

### [8-13] §10 `/dashboard/live` 설명 — "라이브 활동 피드" 부정확 ✅ ACCEPT
- `ui/src/pages/DashboardLive.tsx:42-46` — 제목 "Live agent runs", 설명 "Active runs first, followed by the most recent completed runs".
- 메인 컴포넌트 `ActiveAgentsPanel` (`:50-61`), title "Active / recent".
- **결정**: "라이브 활동 피드" → "활성/최근 에이전트 run 화면", 활동 로그 (`/activity`) 와의 구분 1줄 추가.

## REJECT 후보 — 없음

## DEFER

- 라인 범위 ±1~2 nit
- 영문 `org tree` 같은 용어 취향
- 코드 캡션 문체 (예: "Code N. ..." vs "코드 N. ...")

## 다음 단계

본문 수정 → `2026-05-14-ch08-changes-r1.md` 작성 → r2 재리뷰 요청.
