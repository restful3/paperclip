# ch05 Round 1 — Claude Decisions

_2026-05-24, codex-a (post-clear, handout-2 인용) / 9 findings (모두 ACCEPT)_

## ACCEPT — 모두 (9건)

| # | 위치 | 변경 |
|---|---|---|
| F1 | line 43 (표 1) + line 91 | cloud-upstreams 표 행 — "구현된 라우트 파일/서비스" vs "실제 mounted surface" 분리. line 91 단락에 mount 누락 경고 추가 |
| F2 | line 91 | `cloud_upstreams` 단일 → `cloud_upstream_connections` + `cloud_upstream_runs` 두 테이블 (스키마 `:4-75`) |
| F3 | line 49, 코드 1 트리 | services/ 도메인 트리에 `issues.ts` (201 KB) 추가 + 큰 봉우리 설명에 포함 |
| F4 | line 19 | "모든 mutating activity_log" 단정 완화 — cloud-upstreams 예외 명시 |
| F5 | line 150-159 | middleware 설명 큰 수정 — actor 해석/guard/validation/error 중심으로 정정. 회사 접근 제어는 라우트 내부 helper 에서, redaction 은 logActivity 내부에서 |
| F6 | line 130 | plugin SSE :1363-1423 → :1465-1545 |
| F7 | line 146 | local JWT heartbeat.ts:7649 → :7769-7772 + auth.ts:138-168 추가 |
| F8 | line 136 | "네 표면" → "다섯 credential 경로" (보드 3종 + 에이전트 2종) |
| F9 | line 23 | routes "36개" → ".ts 38개, index/authz 제외 36개" 기준 명시 |

## 변경 적용 파일
- `docs/report/05-server-api.md` (10곳 — F1 이 2곳)

## Round 2 송부 의도

cloud-upstreams mount 누락이라는 큰 인지적 정정 + middleware 표현 큰 수정 포함. FINAL_APPROVAL 또는 미세 1\~2 예상.
