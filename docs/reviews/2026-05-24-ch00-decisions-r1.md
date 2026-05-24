# ch00 Round 1 — Claude Decisions

_2026-05-24, 그룹 A·codex-a / 4 findings 분류_

## ACCEPT (본문 즉시 반영)

### F1 — line 60: 테이블/company_id 카운트 STALE
- **AS-IS**: "80여 개 테이블 중 약 60개가 가진 `company_id` 컬럼"
- **TO-BE**: "88개 테이블 중 67개가 가진 `company_id` 컬럼"
- **근거**: Codex 가 직접 `packages/db/src/schema/*.ts` 에 Perl 스크립트 돌려 카운트 (`tables=88 company_id_tables=67`). `CHANGE-IMPACT-2026-05-23.md:58` 도 88개 기준.
- **검증**: 본문 line 60 직접 Read 완료.

### F2 — line 60: `doc/execution-semantics.md:105` 잘못된 인용
- **AS-IS**: "회복 규칙(stranded reconciliation `:105` · 명시 recovery action `:174-197` · stranded 신호 `:207-245`)"
- **TO-BE**: "회복 규칙(liveness contract `:155-172` · 명시 recovery action `:174-197` · stranded 신호 `:207-240`)"
- **근거**:
  - `:105` 는 user-owned issues 의 "stranded-work reconciliation does not apply to them" — 적용 안 됨 설명이라 출처로 부적절. 직접 Read 확인.
  - "stranded reconciliation" 키워드는 `:105` 와 `:347` 두 곳만 등장. 두 곳 모두 출처로 부적절. Codex 권장대로 단어를 "liveness contract" 으로 바꾸고 `:155-172` 인용 (Non-Terminal Issue Liveness Contract 절).
  - `:207-245` → `:207-240`: 실제 `:241` 부터 `### in_review` 새 섹션 시작. `:207-240` 이 stalled 신호(Agent-assigned `todo`·`backlog`·`in_progress`) 영역으로 정확.

## DEFER

### F3 — ch04 표 헤더 "10종" + grok-local 누락
- **scope 밖 — ch00 본문 변경 X**.
- Codex 검증 결과 ch00 의 어댑터 카운트(13개)는 `registry.ts:511` 과 일치.
- 단, ch04 line 22-37 직접 Read 결과 표 헤더 "**런타임 전용 built-in 10종 + 범용 2종**" + 표 row 에 grok-local 누락 (현재 10+2=12개, 본문은 13개 주장).
- **다음 그룹 A·ch04 라운드에서 처리** — 메모만 기록.

### F4 — line 13 슬로건 인용 위치 누락
- **부가 권장 (Codex 본인이 "다른 문장들과 같은 스타일이면" 으로 약화).**
- 다른 문장 인용 스타일 확인 결과:
  - line 17 `(README.md:131-140)` — 명시
  - line 30 `(server/src/adapters/registry.ts:511-524)` — 명시
  - line 38, 48 등 — 모두 명시
- 슬로건만 인용 위치 없음 — 일관성 측면에서 추가가 자연스러우나, **이번 라운드는 보류**. line 60 의 두 큰 정합성 이슈에 집중. 다음 라운드에서 codex 가 다시 지적하면 그때 ACCEPT.

## REJECT
- 없음.

## 환경 노이즈 (본문 변경 아님)
- 요청 메시지의 `docs/research/00-paperclip-overview.md` 경로는 실제 부재. 리서치 노트는 `01-*~10-*`. 다음 라운드부터 참고 자료 안내 시 정정.

---

## 변경 적용 파일

- `docs/report/00-overview.md` (line 60, 1개 줄에서 두 곳 수정)

## 미적용 (다음 라운드/다른 챕터)

- ch04 표 헤더 + grok-local 표 row 추가 — **ch04 라운드 1 의 1차 후보**
- ch00 line 13 슬로건 인용 — 다음 라운드 codex 가 재지적 시 ACCEPT
