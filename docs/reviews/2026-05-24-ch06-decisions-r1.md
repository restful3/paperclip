# ch06 Round 1 — Claude Decisions

_2026-05-24, codex-b / 18 findings 분류 (16 정합성 + 1 표현 + 1 품질)_

## ACCEPT — 모두 (18건)

Codex 가 모든 지적을 코드 라인 인용으로 자기 검증했고, Claude 가 독립 측정 (`wc -l`, `grep -c <Route`, `ls i18n/locales`, `App.tsx` inbox 라인 grep) 으로 cross-check 완료.

### 정합성 (16건)

| # | 위치 | 변경 | 검증 |
|---|---|---|---|
| F1 | line 19 | 제목 "70+ 페이지" → "라우팅 표 (Route 129개)" | `grep -c "<Route" App.tsx = 129` |
| F2 | line 28 | 회사 설정 경로에 `members`/`cloud-upstream` 추가, `access` legacy 명시 | `App.tsx:73-82` |
| F3 | line 28/46/56 | `Secrets.tsx` 2,155 → 2,444 (3곳, replace_all) | `wc -l = 2444` |
| F4 | line 31 | `IssueRecoveryActionCard.tsx` 537 → 543 | `wc -l = 543` |
| F5 | line 32 | inbox 경로에 `blocked` 추가 | `App.tsx:134-141` 직접 확인 |
| F6 | line 38 | 인스턴스 설정 도메인 라벨에 "(어댑터·플러그인 발췌)" 명시 + 외부 라우트 인라인 표기 | `App.tsx:291-300` |
| F7 | line 52 | "다섯 축" → "여섯 축" | bullet 6개 확인 + line 61 의 "6축" 과 정합 |
| F8 | line 54 | "39개 언어 JSON" → "40개 locale JSON (en 포함)" | `ls locales/*.json = 40` |
| F9 | line 59 | workspace diff 854 → 824 | `wc -l = 824` |
| F10 | line 65/89 | "zod 스키마를 그대로 import" 표현 완화 (typedef 공유 + 일부 응답만 zod) | `client.ts:22-36`, `auth.ts:32-36` |
| F11 | line 111 | 의사코드 `entity_type` → `entityType` | `LiveUpdatesProvider.tsx:650` |
| F12 | line 116 | `plugin.*` 분기 주석 → `activity.logged` 등 실제 처리 채널 | `LiveUpdatesProvider.tsx:817-867` |
| F13 | line 128 | `ui/components/` → `ui/src/components/` | trivially correct |
| F14 | line 130 | "의존성 잠금에서 자유" → "Radix/Tailwind/유틸은 여전히 package.json 관리" | `ui/package.json:49,60,78` |
| F15 | line 130 | brand_color 연결 표현: 회사 아이콘·brand color 입력·일부 회사 선택 UI 색상 표시 | `OnboardingWizard.tsx:107-110` (생성 단계엔 brand_color 없음) |
| F16 | line 159 | "SPEC §10" → "SPEC-implementation §5.1 In Scope" | `SPEC-implementation.md:60-71` In Scope |

### 표현 (1건)

| # | 위치 | 변경 |
|---|---|---|
| 표현 1 | line 57 | "함께 폴리시됐다" → "함께 다듬어졌다" |

### 품질 (1건)

| # | 위치 | 변경 |
|---|---|---|
| 품질 1 | line 67 코드 1 | 디렉터리 트리에 `i18n/` 추가 (context/ 와 hooks/ 사이) |

## REJECT
- 없음.

## DEFER
- 없음.

## 변경 적용 파일

- `docs/report/06-ui-and-board.md` (총 20곳 수정 — 일부는 같은 라인의 다른 부분)

## Round 2 송부 의도

본문 정합성 측면에서 추가 발견은 없을 것으로 기대. `===CODEX_FINAL_APPROVAL===` 기대.
