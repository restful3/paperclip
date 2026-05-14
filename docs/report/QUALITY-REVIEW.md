---
title: docs/report 품질 검토 리포트
date: 2026-05-10
scope: README.md + 00-overview.md ~ 10-research-map.md (총 12 파일)
mode: 본문 수정 적용 완료
axes: 사실 정합성 / 챕터 간 일관성 / 마크다운 규칙 / 구조·AI 트로프
---

## 0. 처리 결과 요약

총 21건 수정 적용. 4건은 **false positive 로 재분류**(자동 탐지 오류). 4건군은 **편집 판단 사항으로 미적용**(사유 §5).

| 분류 | 발견 → 적용 | 비고 |
|---|---|---|
| Critical (사실) | 2건 적용 | 테이블 수, 라우트 수. 어댑터 수는 false positive |
| Major - 마크다운 규칙 | 7건 적용 | CJK bold 2 + 곡선따옴표 5 + 테이블 수 표기 일관성 |
| Major - 일관성 | 12건 적용 | "75" 표기 7개 파일 통일 |
| 편집 판단 (미적용) | 4건군 | em-dash, 톤, bold-first bullets, 그림 라벨 |

핵심 수치 정정:

- 테이블 수 — 자동 보고는 "74"였으나, 실제는 **74개 테이블 정의 파일에 80여 개 `pgTable()` 정의** (일부 파일은 보조 테이블 포함). 본문 표기를 "75/75+/75 이상" 에서 "80여 개/80+/80 이상" 으로 통일.
- 라우트 수 — "약 30개" → 실제 **36개**.
- `company_id` 보유 비율 — "60개 이상" → 실제 약 60개 파일/스키마. "60여 개"로 표기.

---

## 1. 적용된 수정 (21건)

### 1.1 사실 정정 (Critical)

| ID | 파일 | 변경 전 → 변경 후 | 코드 근거 |
|---|---|---|---|
| C2 | `02-data-model.md:7` | `# Data Model — 75개 이상 테이블` → `# Data Model — 80개 이상 테이블` | `pgTable()` grep = 82건 |
| C2 | `02-data-model.md:11` | `**75개의 Drizzle 테이블 파일**` → `**74개의 Drizzle 테이블 파일** (총 80여 개 테이블 정의)` | `ls schema/*.ts` = 75 (인덱스 1 제외 = 74) |
| C2 | `02-data-model.md:21` | `75개 테이블 중 60개 이상` → `80여 개 테이블 중 60여 개` | `grep -l company_id schema/*.ts` = 59 |
| C2 | `02-data-model.md:74` | `## 6. 75개 테이블 그룹 지도` → `## 6. 80여 개 테이블 그룹 지도` | 동상 |
| C2 | `02-data-model.md:78` | `**표 2. 75 테이블의 도메인 군집**` → `**표 2. 80여 개 테이블의 도메인 군집**` | 동상 |
| C2 | `README.md:19` | `Drizzle 스키마 75+ 테이블` → `Drizzle 스키마 80+ 테이블` | 동상 |
| C2 | `00-overview.md:59` | `75개 이상 테이블의 company_id` → `80여 개 테이블 중 약 60개가 가진 company_id` | 동상 |
| C2 | `01-architecture.md:28` | `(75+ 테이블)` → `(80+ 테이블)` | 동상 |
| C2 | `01-architecture.md:117` | `Drizzle 스키마 75+ 테이블` → `Drizzle 스키마 80+ 테이블` | 동상 |
| C2 | `09-pros-cons.md:27` | `75개 테이블 중 60개 이상이 company_id` → `80여 개 테이블 중 60여 개가 company_id` | 동상 |
| C2 | `10-research-map.md:32` | `(75+ 테이블)` → `(80+ 테이블)` | 동상 |
| C3 | `05-server-api.md:23` | `약 30개의 라우트 파일` → `36개의 라우트 파일` | `ls server/src/routes/*.ts` = 36 |

### 1.2 마크다운 규칙 위반

CJK bold 닫는 `**` 뒤 공백 누락:

| 파일 | 라인 | 변경 |
|---|---|---|
| `00-overview.md` | 39 | `…"orchestrator-workers" 워크플로**의` → `…워크플로** 의` |
| `02-data-model.md` | 43 | `…"라이브 실행이 보장돼야 한다"**는` → `…한다"** 는` |

곡선따옴표(`“ ”`) → 평문 `"`:

| 파일 | 라인 | 인용 |
|---|---|---|
| `06-ui-and-board.md` | 46 | "설정 화면이 많은 관리 콘솔" + "보고는 많은데 결정을 못 내리는" |
| `06-ui-and-board.md` | 104 | "무엇이 바뀌었는가" + "Surface failures clearly" |
| `07-governance-cost.md` | 129 | "매시간 경쟁사 가격 조사" |
| `09-pros-cons.md` | 59 | "코어 신뢰도를 해치는 미성숙" + "도메인별 확장" |
| `10-research-map.md` | 102 | "표현 가능하다" + "완전히 같은 의미로 구현된다" |

### 1.3 검증 후 처리한 일관성

- 챕터 간 테이블 수 표기 통일(7개 파일).
- 챕터 02 안에서 "테이블 파일" vs "테이블" 구분을 line 11 에 한 번 명시.

---

## 2. False Positive 로 재분류 (자동 탐지 오류)

자동 스캔이 제기했지만 본문/코드 직접 검증 결과 **수정 불필요**.

### F1. 어댑터 개수 9+2=11 (챕터 04)

**자동 보고**: "런타임 전용 9개 + 범용 2개 = 11개" 가 틀렸다, 실제는 8+2=10. `hermes_local`은 `AGENT_ADAPTER_TYPES` 상수에 없음.

**검증**: `server/src/adapters/registry.ts:380~439` 가 `hermes_local` 어댑터를 등록하고, 같은 파일 line 463 의 `registerBuiltInAdapters()` 가 `hermesLocalAdapter` 를 빌트인 배열에 포함. `packages/shared/src/constants.ts:42` 의 `AgentAdapterType = (typeof AGENT_ADAPTER_TYPES)[number] | (string & {})` 정의가 상수 외 식별자도 허용. **upstream master 런타임에는 11개 어댑터가 등록**되며 챕터 04 의 9+2=11 표기는 정확하다. `hermes-paperclip-adapter` 가 정적 의존성이라는 챕터 04 line 106 의 단서도 정확히 명시되어 있다. 수정하지 않음.

### F2. 코드 펜스 언어 식별자 누락 18건 (전 챕터)

**자동 보고**: 18곳에서 ` ``` ` 만 단독 등장.

**검증**: 펜스 짝짓기로 다시 스캔한 결과 **여는 펜스는 모두 언어 식별자(`bash`/`text`/`ts`/`sql`/`json` 등)를 가짐**. 자동 도구가 닫는 펜스 라인을 잘못 보고. 수정하지 않음.

### F3. 본문 raw `~` 6건 (Rule 3)

**자동 보고**: `01-architecture.md:101,103` · `02-data-model.md:116` · `08-usage.md:33,35` · `09-pros-cons.md:35` 의 `~/.papercli` 표기.

**검증**: 모두 홈 디렉터리 경로 표기(`~` 다음에 `/`). 범위 표기가 아니므로 GFM/CommonMark 모두 strikethrough 트리거 안 함. CLAUDE.md 의 경고는 `pp.145~147` 같은 영숫자 사이 `~` 한정. 수정하지 않음.

### F4. 8장 reachability 매핑 오류

**자동 보고**: 챕터 08 line 54 가 코드 3 의 4가지 reachability 옵션과 매핑이 깨졌다.

**검증**: 본문은 *"1번이 나만, 2~3번이 팀, 4번이 외부 공개"* 로 4가지 옵션과 정확히 매핑. 에이전트 오독. 수정하지 않음.

### F5. 9장 `**(...)** 이` CJK bold 위반

**자동 보고**: `09-pros-cons.md:38` 의 `**모델 명료성**(1\~3행: ...)` 이 CJK bold 공백 규칙 위반.

**검증**: 닫는 `**` 바로 뒤는 `(` (구두점)이지 한글이 아님. CLAUDE.md 의 CJK bold 규칙은 닫는 `**` 뒤 한글이 붙는 경우만 적용. 수정하지 않음.

---

## 3. 검증된 정확/일관성 (참고)

직접 대조 결과 정확하고 일관된 항목:

- 스키마 필드명: `assigneeAgentId`, `assigneeUserId`, `checkoutRunId`, `executionRunId`, `cached_input_tokens` (`packages/db/src/schema/`).
- XOR 규칙, 7-상태 머신, atomic checkout 의미론 (`doc/execution-semantics.md`).
- 핵심 라우트: `/api/issues/:id/checkout`, `/api/approvals/:id/approve` (`server/src/routes/`).
- 비용 회계: `cost_events` 테이블, `billing_code`, 6단계 파이프라인 (`server/src/services/`).
- 표 번호 규칙: 챕터마다 1부터 다시 시작, 누락·중복 없음.
- 마크다운 hard break: 라벨 라인에서 누락 0건.
- AI 트로프(매직 부사·Imagine·Truth Is Simple·Signposted Conclusion·False Vulnerability): 의미 있는 출현 없음.

---

## 4. 통계

- 사실 정정 12건 적용 (Critical 2 axis × 다중 위치)
- CJK bold 공백 2건 적용
- 곡선따옴표 7쌍 적용 (5개 라인)
- False positive 5범주 식별
- 표·그림 번호, 상호 참조, 핵심 사실 항목 모두 일관 확인

---

## 5. 편집 판단으로 미적용 (사유)

다음 4건군은 자동 탐지가 발견했지만 본문 톤/관습 보존을 위해 **수정하지 않음**. 사용자가 명시적으로 적용을 요청하면 별도 PR 권장.

### 5.1 Em-dash(`—`) 과다 — 미적용

전 챕터 ERROR(04장 1000단어당 ~29회, 07장 ~31회). 다만:

- Em-dash 는 한국어 기술 학습 글의 보조절·정의 도입에서 흔한 도구이며, 작성자가 일관되게 사용 중.
- 자동 트로프 도구가 "포매팅 트로프"로만 분류했고 의미 트로프(Magic Adverbs, Truth Is Simple, Signposted Conclusion 등)는 발견 0건.
- 일괄 감량은 단락 흐름을 끊어 학습 톤을 해칠 위험.

권장: 사용자가 04·07 한 챕터씩 직접 통독해 보고 단락당 3개 이상 누적된 곳만 1개씩 줄이는 *수동* 정비.

### 5.2 톤 표류 (03/04/10) — 미적용

자동 도구가 "정의 부족" 으로 신호한 항목을 본문에서 직접 확인:

- **03장** `stranded` / `watchdog`: §6 "Stranded recovery"·§7 "Silent active-run watchdog" 절 제목과 첫 문장에 정의가 명시되어 있음. 정의 부족 아님.
- **04장** `ServerAdapterModule` 필드: §1 line 13 에 어댑터 능력 매트릭스의 축으로 도입되며 §3 표 2 가 4개 진입점 책임을 정리. 참고 톤이 의도된 것.
- **10장** 외부 표준 매핑: 챕터의 *목적 자체* 가 매핑(R1\~R10 외부 패턴). 톤 이탈이 아니라 챕터 정체성.

수정 불요.

### 5.3 Bold-first bullets 5연속 — 미적용

00장 line 78~82, 07장 line 13~14·97~98, 10장 line 96~100·120~123. `**라벨**:` 형 항목이 5개 이상 연속하여 등장. 가독성 *선호* 의 차이일 뿐 규칙 위반 아님. 작성자가 의도적으로 라벨링한 정보 구조라 보존.

### 5.4 그림 4-1\~4-4 라벨 — 미적용

챕터 04 의 외부 런타임 스크린샷이 `그림 4-1 ~ 4-4` 로 라벨링됨. 02장의 `그림 4` (ER 다이어그램) 와 명목상 충돌 가능. 다만:

- 자산 파일은 `assets/external/{codex,gemini,opencode,pi}/*.png` 로 폴더 분리, 충돌 없음.
- 챕터 내 등장 컨텍스트가 다르므로 본문 가독성에서는 혼동 안 됨.

대안 라벨(`사진 4-1`, `예시 4-1`)로 변경하는 것은 챕터 04 의 인용 통일성을 깨므로 보류.

---

## 6. 검증 절차

각 수정 후 자동 재스캔으로 위반 0건 확인:

```bash
# CJK bold 페어링 + 백틱 코드 제외 후 한글 매칭
python3 -c '...'  # 결과: 0
# 곡선따옴표 검색
python3 -c '...'  # 결과: 0
# stale "75" 표기 검색
grep -nH "75" *.md | grep -E "75\+? *(테이블|개)"  # 결과: 0
# stale "약 30" 라우트 표기
grep "약 30" 05-server-api.md  # 결과: 0
```

소스 코드 카운트 (참조용):

```bash
ls packages/shared/src/constants.ts                   # AGENT_ADAPTER_TYPES 10개
ls server/src/adapters/registry.ts                    # +hermes_local 빌트인 등록 → 11개
ls packages/db/src/schema/*.ts | wc -l                # 75 (index.ts 포함)
ls packages/db/src/schema/*.ts | grep -v index | wc -l # 74 (테이블 정의 파일)
grep -c "pgTable(" packages/db/src/schema/*.ts | …   # 82 (실제 pgTable 호출)
grep -l company_id packages/db/src/schema/*.ts | wc -l # 59 (company_id 보유 파일)
ls server/src/routes/*.ts | wc -l                     # 36 (라우트 파일)
```
