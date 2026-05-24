# ch02 Round 1 — Claude Decisions

_2026-05-24, codex-a (post-clear, handout 인용) / 10 findings 분류_

## ACCEPT (9건)

| # | 위치 | 변경 |
|---|---|---|
| F1 | line 11 | 86개/78개 → 88개/79개 (Group A 합의 표준) |
| F2 | line 17, 21 | "모든 비즈니스 데이터" → "핵심 회사 도메인 데이터" + 80여/60여 → 88/67 + 플러그인 예외 |
| F3 | line 34 | `requested_by_agent_id?` ⊕ `_user_id?` → 콤마로 분리 (XOR 불변조건 없음) |
| F4 | line 47 | SPEC §8 "affected_rows == 1" → SPEC-implementation §10.4.1 atomic checkout contract |
| F5 | line 82 | `cloud_upstreams` → `cloud_upstream_connections`, `cloud_upstream_runs` (실제 2 테이블) |
| F6 | line 89 | "디렉터리 구조와 1:1" → "파일명·서비스 경계와 대체로 대응" |
| F8 | line 116 | embedded-postgres pin 표현 정확화 + PostgreSQL 17 단언 약화 |
| F9 | line 82 | `users` (Better Auth), `auth.*` → 실제 테이블명 `user/session/account/verification` |
| F10 | line 37 (표현) | "의미심장하다" → 중립 표현 |

## DEFER (1건)

### F7 — `docs/research/06-drizzle-pglite.md:40` 의 86/78 → 88/79
- **이유**: research scope 밖. ch02 본문은 이번 변경으로 정합.
- **처리**: 합의 루프 후 별도 작업.

## 변경 적용 파일

- `docs/report/02-data-model.md` (9곳)

## Round 2 송부 의도

Group A 합의 표준 (88/67) 본문 반영 완료. 큰 변경 (atomic checkout contract 인용 정정, cloud upstream 2 테이블 분리) 포함. FINAL_APPROVAL 또는 미세 정합 1\~2 예상.
