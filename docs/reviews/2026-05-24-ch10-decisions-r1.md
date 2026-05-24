## ch10 Round 1 — Claude Decisions

_2026-05-24, codex-b / 8 정합성 + 1 품질 findings (모두 ACCEPT)_

## ACCEPT (9건)

| # | 위치 | 변경 |
|---|---|---|
| F1 | line 32 | R6 행: `packages/db/* (80+ 테이블 · 마이그레이션)` → `packages/db/* (pgTable 88개 · company_id 보유 67개 · schema 파일 79개 · 마이그레이션)` (그룹 B handout 표준) |
| F2 | line 33 | R7 행: `24개 워크스페이스 패키지 (adapters 9 · ...)` → `26개 워크스페이스 패키지 (packages/adapters 10개: acpx-local, claude-local, codex-local, cursor-cloud, cursor-local, gemini-local, grok-local, openclaw-gateway, opencode-local, pi-local · plugins · server · ui · cli 등; sandbox-providers 5종은 workspace 제외)` |
| F3 | line 35 | R9 행: `startup/periodic reconciliation 7-pass (server/src/index.ts:676-780)` → `startup reconciliation 7단계(:723-760) + periodic recovery loop(:789-826)` (verified: startup `reapOrphanedRuns` :726, periodic `reapOrphanedRuns` :792) |
| F4 | line 38 | 표 1 순서 설명 수정: `표 1의 행 순서는 *외부 인프라(R6-R9) → 오케스트레이션 패턴(R1, R10) → 실행 런타임(R2-R5)*` (잘못) → `표 1은 리서치 번호순으로 배열되어 있고, 각 행을 그림 10의 4-층 위계에 다시 대응시켜 읽을 수 있다.` |
| F5 | line 44 | `Anthropic의 "Building Effective Agents" (Schluntz & Zhang)가 정의한 4가지 빌딩 블록` → `Anthropic "Building Effective Agents" taxonomy 중 Paperclip과 직접 연결되는 네 도식(augmented LLM · orchestrator-workers · evaluator-optimizer · autonomous agent)` (전체 taxonomy 과소 열거 회피) |
| F6 | line 52 | `이 블록이 곧 Paperclip의 "회사" 모델` (구현 정합성상 과함) → `이 블록은 Paperclip의 "회사" 모델과 가장 가까운 외부 패턴이다. 운영 원칙상 CEO/PM 중심 분해로 쓸 수 있고, reports_to 보고 라인은 책임 추적 표면이 되며, control plane은 분배·감사·비용 귀속 인프라를 제공한다.` (ch08 합의: CEO 자동 분해는 SPEC 목표) |
| F7 | line 123 | `ServerAdapterModule.execute / testEnvironment 2 필수 메서드 (packages/adapter-utils/src/types.ts:349-431)` (옵션 필드 포함) → `2 필수 메서드(:349-352) + 옵션 필드(listSkills ... getRuntimeCommandSpec, :353-431)` |
| F8 | lines 135-141 | 검증 시나리오 7단계: `/companies` 진입 → `/onboarding` (no-company 흐름이 onboarding 으로 리다이렉트, `ui/src/App.tsx:200-218`, `ui/src/lib/onboarding-route.ts:46-50`); `/agents/new` 별도 단계 → onboarding step 2 에 CEO hire 통합; `activity_log` → `/<prefix>/activity` |
| Q1 | line 17 | `바깥 4개 층에 정의된 표준/패턴/런타임` (Paperclip 코어를 포함하는 4층과 충돌) → `앞의 네 층에 배치된 표준/패턴/런타임/코어의 조합` |

## REJECT / DEFER
없음.

## 변경 적용 파일
- `docs/report/10-research-map.md` (9곳)

## Round 2 송부 의도
FINAL_APPROVAL 기대. F3 라인 범위는 Codex 인용을 그대로 채택(±2-3 라인 정확).
