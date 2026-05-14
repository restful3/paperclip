# Paperclip 학습 리포트 후속 사이클 — 최종 요약

**기간**: 2026-05-14 (직전 사이클 종료 후 후속)
**범위**: HANDOFF.md §7 후보 1·2·3 일괄 진행
**상태**: ✅ 모든 단계 `===CODEX_FINAL_APPROVAL===`

---

## 1. 결과 한 단락

이전 사이클 종료 후 남은 3 후속 작업 — (1) 챕터 ↔ 리서치 cross-reference 정합성 검토, (2) DEFER 항목 일괄 정합화, (3) PDF 재빌드 — 를 순서대로 진행했다. Phase 1 에서 codex r1 이 substantive 불일치 3건 (Claude billing label 레이어 구분 / Gemini adapter 호출 형태 / Plugin SSE bridge 라인 범위) 을 잡았고 모두 ACCEPT 후 적용, r2 에서 `CODEX_FINAL_APPROVAL`. Phase 2 에서 그림 번호 충돌 3쌍 (그림 2 Ch00↔Ch01 / 그림 5 Ch02↔Ch03 / 그림 6 Ch02↔Ch03) 을 수술적 chapter-prefix 이동 (Ch01 그림 2 → 1-1, Ch03 그림 5/5-A/6 → 3-1/3-1-A/3-2) 으로 해소하고 한국어 조사 일치 3건도 동시 정정, r1 에서 `CODEX_FINAL_APPROVAL`. Phase 3 slideflow 빌더로 PDF 재컴파일 완료 (7.06 MB). Paperclip 코드 변경 0건.

## 2. Phase 별 결과

### Phase 1 — Cross-reference 정합성 (R02·R03·R04·R09 ↔ Ch03·Ch04·Ch05·Ch07)

라운드 2 까지 진행. r1 substantive 3건 / r2 최종 승인.

| ID | 위치 | 사실 충돌 | 진실 소스 | 수정 |
|---|---|---|---|---|
| B-1 | R03 §5 line 75 | adapter billing label 과 ledger 정규화 값의 레이어 구분 부재 | `claude-local/execute.ts:119`, `heartbeat.ts:1275-1285`, `shared/constants.ts:484-491` | R03 §5 에 한 문장 보강 |
| C-1 | Ch04 §2 표 1 line 33 + R04 §4 line 51 | `gemini-local` 호출 형태가 공식 `gemini -p` 와 달리 `--prompt + stream-json` | `gemini-local/execute.ts:506-518`, `:448` | Ch04 표 + R04 §4 분기 명시 |
| D-1 | Ch05 §4 line 125 | Plugin SSE bridge 라인 범위 `1373-1399` 가 헤더 직전에서 끝나 stream delivery 누락 | `plugins.ts:1363-1423` | Ch05 §4 인용 범위 정정 |

쌍 A (R02 §5 ↔ Ch04 §6) 는 라운드 1 에서 불일치 0건. 수정 없음.

리뷰 산출물:
- `2026-05-14-xref-codex-r1.md` (7.0 KB)
- `2026-05-14-xref-decisions-r1.md`
- `2026-05-14-xref-changes-r1.md`
- `2026-05-14-xref-codex-r2.md` (FINAL_APPROVAL)

본문 수정 4 파일: `docs/research/03-claude-code-cli.md`, `docs/research/04-headless-coding-agents.md`, `docs/report/04-adapters-and-skills.md`, `docs/report/05-server-api.md`.

### Phase 2 — DEFER 정합화 (그림 번호 충돌 + R09 `~` 이스케이프)

라운드 1 단발. r1 FINAL_APPROVAL.

**그림 번호 충돌 (3쌍 해소)**:

| 충돌 | 해소 방법 |
|---|---|
| 그림 2 (Ch00 Orchestrator-workers ↔ Ch01 pnpm 모노레포 지도) | Ch01 → **그림 1-1** |
| 그림 5 (Ch02 Drizzle Studio ↔ Ch03 heartbeat 시퀀스) | Ch03 → **그림 3-1** |
| 그림 5-A (Ch03) | Ch03 → **그림 3-1-A** (chapter-prefix 호환) |
| 그림 6 (Ch02 PGlite ↔ Ch03 issue 7-state) | Ch03 → **그림 3-2** |

원칙: Ch00 anchor (그림 1, 그림 2) 와 unique 챕터 (Ch02, Ch04, Ch05, Ch06, Ch07, Ch08, Ch10) 의 라벨은 손대지 않음. 충돌 챕터 (Ch01, Ch03) 만 chapter-prefix 로 이동 — 기존 Ch04 (4-1\~4) / Ch06 (6-1) / Ch10 (10-1\~9) 패턴과 일관.

한국어 조사 일치 (3건): `1-1는→은`, `3-1가→이` (자음 종성), `3-2 이→가` (모음 종성). Ch10 의 기존 `그림 10-1 ... 은` / `그림 10-2 ... 는` 패턴과 동일한 규칙.

**R09 `~` 이스케이프**:

`docs/research/09-sse-heartbeat-patterns.md:33` 의 `k=2\~3` 은 이미 escape 처리되어 있어 추가 변경 없음 (점검만).

리뷰 산출물:
- `2026-05-14-defer-decisions-r1.md`
- `2026-05-14-defer-codex-r1.md` (FINAL_APPROVAL)

본문 수정 2 파일: `docs/report/01-architecture.md`, `docs/report/03-runtime-execution.md`.

### Phase 3 — PDF 재빌드

`docs/report/pdf/README.md` 가 정의한 재빌드 절차로 진행:

1. `docs/report/_pdf-build/` 의 11개 챕터 사본 갱신 (`sed 's|assets/|figs/|g'` 적용)
2. `_pdf-build/figs/` 재동기화 (assets → figs)
3. slideflow `build.py` 실행 (`--theme research --brand ai-odyssey --pdf`)
4. 산출물 `docs/report/pdf/` 로 복사

**최종 산출물**:
- `paperclip-learning-report.pdf` — 7,059,363 bytes (이전 6,859,317 bytes, +200 KB)
- `paperclip-learning-report.html` — 211,544 bytes (이전 181,657 bytes)
- `figs/` 28개 자산 동기화

**HTML 내 변경 반영 검증** (grep 기반):

```
그림 1-1 / 그림 3-1 / 그림 3-1-A / 그림 3-2  (신규 라벨, Phase 2)
gemini --output-format stream-json --prompt  (Phase 1 C-1)
plugins.ts:1363-1423                          (Phase 1 D-1)
```

잔존 `>그림 2.`, `>그림 5.`, `>그림 6.` 은 Ch00 Orchestrator-workers + Ch02 Drizzle Studio + Ch02 PGlite 의 의도적 유지분.

## 3. 누적 통계

| 지표 | 값 |
|---|---|
| 변경 본문 파일 | 6 (`docs/report` 4 + `docs/research` 2) |
| codex 라운드 | 4 (Phase 1: r1+r2, Phase 2: r1, Phase 3: 검증 grep) |
| substantive ACCEPT | 3 (Phase 1 만 — Phase 2 는 자체 결정 후 1라운드 승인) |
| REJECT | 0 |
| DEFER 신규 | 0 |
| Paperclip 코드 변경 | 0 |

## 4. 합의된 사실 변화 (이전 핸드오프 §3 보강)

이전 사이클의 §3 합의된 사실 묶음은 그대로 유효. 본 사이클이 추가로 정리한 사항:

**Cost / billing**:
- Claude adapter result label `"api" | "subscription" | "metered_api"` 는 **pre-ledger 값**. `cost_events` ledger 에는 heartbeat 서비스가 `api → metered_api`, `subscription → subscription_included` 로 정규화 (`server/src/services/heartbeat.ts:1275-1285`).
- `BILLING_TYPES` 6 값: `metered_api, subscription_included, subscription_overage, credits, fixed, unknown` (`packages/shared/src/constants.ts:484-491`).

**Adapter invocation**:
- `gemini-local` 은 공식 `gemini -p "..."` 와 달리 `--prompt + --output-format stream-json + --approval-mode yolo` 사용 (`packages/adapters/gemini-local/src/server/execute.ts:506-518`).

**Plugin SSE bridge**:
- SSE route 전체 범위: `server/src/routes/plugins.ts:1363-1423`. 핵심 stream fan-out 콜백은 `:1407-1423`.

## 5. 후속 사이클 후보 (HANDOFF.md §7 갱신)

후보 1·2·3 완료. 남은 후보:

4. **Paperclip 코드 변경 후 stale 검사** — 본 사이클은 2026-05-14 코드 기준 검증. 이후 코드 변경 발생 시 §3 + 본 §4 합의 사실 재검증 필요.
5. **신규 챕터 / 리서치 추가** — 사용자가 도메인 지정 시.

## 6. 새 세션 시작 프롬프트 (그대로 입력)

> Paperclip 학습 리포트 합의 루프 후속 작업을 진행해줘. 직전 사이클 (cross-reference + DEFER 정합화 + PDF 재빌드) 은 모두 종료된 상태야 — 핸드오프는 `/media/restful3/data/workspace/paperclip/docs/reviews/HANDOFF.md`, 후속 사이클 결과는 `2026-05-14-followup-cycle-final-summary.md` 에 있다. 합의된 사실 묶음 (HANDOFF §3 + final-summary §4) · 운영 관습 (HANDOFF §5) · 새 후속 사이클 후보 (final-summary §5) 를 그대로 따른다. codex peer reviewer 스킬 사용, tmux 타깃 `paperclip:codex`, 자율 모드 (중간 보고 없이 끝나면 일괄 보고). 진행할 작업: [final-summary §5 의 후보 중 하나 선택, 또는 새 지시].
