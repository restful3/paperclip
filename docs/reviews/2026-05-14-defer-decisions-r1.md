# DEFER 정합화 — 결정 로그 r1

라운드 자체적 결정 (codex r1 검토 전에 적용한 변경).

## 1. 외부 인용 그림 글로벌 번호 충돌 — 해소

### 발견된 충돌

| 번호 | 충돌 |
|---|---|
| 그림 2 | Ch00 (Orchestrator-workers) ↔ Ch01 (pnpm 모노레포 지도) |
| 그림 5 | Ch02 (Drizzle Studio) ↔ Ch03 (heartbeat 시퀀스) |
| 그림 6 | Ch02 (PGlite) ↔ Ch03 (issue 7-state) |

### 수정 전략 (수술적)

**원칙**: Ch00 의 anchor figure (그림 1, 그림 2) 는 그대로 유지. 충돌 챕터 (Ch01, Ch03) 만 chapter-prefix 로 이동. 충돌 없는 다른 챕터는 손대지 않음.

이는 다음 두 가지 이유로 정당화됨:
1. **수술적 변경 원칙** — 최소 blast radius. 5 파일 → 2 파일로 축소.
2. **기존 부분 패턴과 일관** — Ch04 (4-1\~4), Ch06 (6-1), Ch10 (10-1\~9) 이 이미 chapter-prefix 사용 중. Ch01·Ch03 도 같은 패턴으로 통합.

### 변경 매핑

| 파일 | AS-IS | TO-BE |
|---|---|---|
| Ch01 (`01-architecture.md`) | 그림 2 (pnpm map) | 그림 1-1 |
| Ch03 (`03-runtime-execution.md`) | 그림 5 (heartbeat seq) | 그림 3-1 |
| Ch03 | 그림 5-A (HeartBeat 패턴) | 그림 3-1-A |
| Ch03 | 그림 6 (issue 7-state) | 그림 3-2 |

### 한국어 조사 일치 (3건)

번호 끝 한글 음가에 따른 조사 변경 (자음/모음 종성):

| 위치 | AS-IS | TO-BE | 사유 |
|---|---|---|---|
| Ch01 line 11 | `그림 1-1는` | `그림 1-1은` | `1=일` 자음 종성 |
| Ch03 line 11 | `그림 3-1가` | `그림 3-1이` | `1=일` 자음 종성 |
| Ch03 line 53 | `**그림 3-2** 이` | `**그림 3-2** 가` | `2=이` 모음 종성 |

(Ch10 의 기존 `그림 10-1 (...) 은`, `그림 10-2 (...) 는`, `그림 10-3 (...) 은`, `그림 10-4 (...) 는` 패턴이 이미 같은 규칙을 적용 중이라 새 변경은 그 규칙과 일관)

### 손대지 않은 그림 번호 (충돌 없음, 유지)

- Ch00 그림 1, 그림 2 — anchor 유지
- Ch02 그림 4, 그림 5, 그림 6 — Ch03 가 이동했으므로 이제 unique
- Ch04 그림 7, 그림 4-1\~4 — 기존 패턴 유지
- Ch05 그림 3, Ch06 그림 6-1, Ch07 그림 9, Ch08 그림 8 — 각각 unique
- Ch10 그림 10, 그림 10-1\~9 — 기존 패턴 유지

### 손대지 않은 meta-tracking 파일

`docs/report/CHANGE-IMPACT-2026-05.md` 와 `docs/report/QUALITY-REVIEW.md` 의 figure 참조는 *과거 시점 audit log* 이므로 손대지 않음. 본 사이클의 검토 범위는 사용자 facing 본문 (Ch00\~Ch10) 한정.

## 2. R09 본문 `~` 이스케이프 — 이미 적용됨 (점검 완료)

- `docs/research/09-sse-heartbeat-patterns.md:33` 의 `k=2\~3` 이미 escape 처리 완료. 추가 변경 없음.
- 다른 research/report 본문 unescaped `~` 사례: `docs/research/06-drizzle-pglite.md:19` 의 `~31KB` — single `~` (approximation 의미, paired 매칭 없음, strikethrough 미 트리거). 변경 불필요.

## 다음 단계

위 변경을 codex r1 으로 검토 요청. 새 substantive 사실 오류 없으면 r1 즉시 `===CODEX_FINAL_APPROVAL===` 기대 (Phase 1 과 달리 본 변경은 사실 인용이 아니라 라벨 정합성이므로 substantive 오류 가능성은 낮음).
