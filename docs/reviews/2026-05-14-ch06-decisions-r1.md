# 챕터 06 — 라운드 1 결정 로그

## ACCEPT (5)
- A1 `:11` UI 스택 정정 — React 19, react-router-dom 7, Vite 6, Tailwind CSS 4, TanStack Query 5
- A2 `:25-42` 라우팅 표 재작성 — `/my-issues`, `/adapters`, `/plugin-settings/*`, `/issue-chat-ux-lab`, `/invite-ux-lab` 등 stale 라우트 제거. `App.tsx:61-138` 실제 라우트 트리 기준. 어댑터 매니저는 `/instance/settings/adapters`, 플러그인은 `/plugins/:pluginId` + `/:companyPrefix/:pluginRoutePath/*`. `/skills/*`, `/inbox/*`, `/goals/*`, `/u/:userSlug` 등 추가
- A3 `:28, 44` 회사 설정 행에 `/company/settings/secrets` + `Secrets.tsx` (2,155 LOC) 추가. 복잡도 문단에도 반영
- A4 §3 `:48-67` 사이드바 분할 — `Sidebar.tsx` 가 `SidebarSection`/`SidebarAgents`/`SidebarProjects` 를 조합한다고 명시
- A5 §2 또는 새 단락에 `IssueRecoveryActionCard.tsx` (537 LOC) — Ch05 recovery-actions API의 UI 표면

## 누적 LOC
- Ch06 r1 예상 변경: ~25 LOC. 전체 누적: ~235 / 200 — 한참 초과. 사용자 일괄 보고 모드라 final-summary에서 정리.
