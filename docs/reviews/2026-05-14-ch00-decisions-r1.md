# 챕터 00 — 라운드 1 결정 로그

검증한 ACCEPT/REJECT/DEFER 분류. 모든 ACCEPT 항목은 Paperclip 코드를 직접 읽어 사실을 확인했다.

## ACCEPT (9개)

### A1. README 문제 목록 5개 → 6개 (기준 1)
- 위치: `00-overview.md:17,19-23`
- 검증: `README.md:131-140` 의 "Problems Paperclip solves" 표는 6행. 6번째 행은 "아이디어→repo→Claude Code→babysit". `doc/GOAL.md` 는 enumeration 형태가 아니므로 인용 부적절.
- 수정: "README와 `doc/GOAL.md`" 인용을 "README(`README.md:131-140`)"로 줄이고, 목록을 6개로 확장.

### A2. built-in adapter 카운트 11 → 12 (기준 1)
- 위치: `00-overview.md:29`
- 검증: 
  - `server/src/adapters/registry.ts:480-493` → 12개 등록 (acpx_local, claude_local, codex_local, opencode_local, pi_local, cursor_cloud, cursor (cursorLocalAdapter), gemini_local, openclaw_gateway, hermes_local, process, http)
  - `server/src/adapters/builtin-adapter-types.ts:4-17` → 12개 동일
  - `packages/shared/src/constants.ts:30-42` AGENT_ADAPTER_TYPES → 11개 (hermes_local 없음). 셀러 메타로만 사용.
- 수정: "11개 / 9 runtime + 2 generic / Hermes 포함" → "12개의 built-in adapter type — 10개 구체 런타임 + `process`/`http` 2개 generic" + `registry.ts:480-493` 인용. cursor-cloud 신규 사실 포함.

### A3. 어댑터 계약 invoke/status/cancel → 실 구현 execute/testEnvironment (기준 1)
- 위치: `00-overview.md:37` (vs `00-overview.md:35`에서 본문은 이미 "execute / testEnvironment / …"라 적었음 — 내부 모순)
- 검증: `packages/adapter-utils/src/types.ts:349-389` `ServerAdapterModule` 의 필수 메서드는 `execute` + `testEnvironment`. invoke/status/cancel은 `doc/SPEC.md:207-215`의 장기 설계.
- 수정: 37줄을 "장기 SPEC(`doc/SPEC.md:207-215`)은 `invoke/status/cancel`을 최소 계약으로 그리지만, 현재 구현(`packages/adapter-utils/src/types.ts:349-352`)은 `execute`와 `testEnvironment`를 필수로 둔다."로.

### A4. issue → goal 도달은 원칙이지 불변식 아님 (기준 1)
- 위치: `00-overview.md:56` (표 1의 원칙 #4 본문)
- 검증: `packages/db/src/schema/issues.ts:28-29` `goalId` / `parentId` 둘 다 nullable. 표 1 자체(원칙 슬로건)는 그대로 두고, 코드 매핑 단락(59줄)에서 톤만 낮춤.
- 수정: 59줄 "4는 부모/자식 위계로(`issues.parent_id` self-FK)" → "4는 부모/자식 위계로(`packages/db/src/schema/issues.ts:28-29` — `goal_id`/`parent_id`는 nullable이며 fallback 서비스가 보장)".

### A5. "PRODUCT와 SPEC이 일관되게 강조하는 다섯 원칙" 표현 부정확 (기준 1)
- 위치: `00-overview.md:47`
- 검증: 
  - `doc/PRODUCT.md:61-71` → PRODUCT는 5개 원칙(Unopinionated / Company / **Adapter config defines the agent** / All work / **Control plane, not execution plane**)
  - `doc/SPEC.md:521-531` → SPEC는 9개 원칙
  - 리포트 표 1 5개는 PRODUCT와 SPEC을 섞은 *학습용 추림*
- 수정: "PRODUCT.md(`doc/PRODUCT.md:61-71`)의 다섯 제품 원칙과 SPEC.md §13(`doc/SPEC.md:521-531`)의 아홉 가지 consolidated principles를 학습용으로 다섯 축으로 추리면 다음과 같다."로.

### A6. V1 Out 전체가 plugin 영역이라는 단정 (기준 1)
- 위치: `00-overview.md:72`
- 검증: 
  - `doc/SPEC.md:515-517` → KB, 매출 회계는 "future plugin"으로 명시
  - `doc/SPEC-implementation.md:75-82` V1 Out 6개 중 multi-board governance / self-healing은 plugin이 아닌 거버넌스/오케스트레이션 확장
- 수정: "*Out* 열은 곧 *플러그인이 메우는 영역*이다" → "Out 중 KB · 매출 회계 · 공개 마켓플레이스는 future plugin으로 문서화되어 있고(`doc/SPEC.md:515-517`), multi-board governance와 self-healing orchestration은 V1 밖의 거버넌스/오케스트레이션 확장으로 남아 있다(`doc/SPEC-implementation.md:75-82`)."

### A7. README 페르소나 5 → 7 (기준 1)
- 위치: `00-overview.md:76-82`
- 검증: `README.md:70-78` → 7개 (자율 회사 빌더, 다중 에이전트 조정자, 20-탭, 24/7 + 감사, **비용 모니터링/예산**, **태스크 매니저 같은 프로세스**, 모바일)
- 수정: "다섯 가지" → "일곱 가지" + 누락 2개 추가 + `README.md:70-78` 인용.

### A8. 그림 1-A 본문 참조 굵음 제거 (기준 4, 좁힘)
- 위치: `00-overview.md:39`
- 검증: 캡션은 "**그림 N. 〈제목〉**" 형식 (관습). 본문 참조까지 굵게 쓰면 캡션과 시각적 충돌. `1-A`라는 보조 번호 자체는 외부 인용 그림을 메인 번호와 분리하려는 의도적 선택으로 유지.
- 수정: "`**그림 1-A**가 그 원형이다`" → "그림 1-A가 그 원형이다" (평문).

### A9. 핵심 사실 단락에 path:line 보강 (기준 5, 좁힘)
- 위치: 위 A1\~A7 수정과 함께 line citation 동시 삽입.
- 모든 문장마다는 안 함 (가독성 해침). 수치/리스트/계약/원칙 같은 검증 가능한 사실에만.

## REJECT (1개)

### R1. CJK bold 뒤 공백 광범위 적용 (Codex 기준 4 지적)
- Codex가 지적한 위치: `:11, :13, :25, :29, :35, :39, :59, :72` 다수
- 근거: 사용자 정의 마크다운 규칙(`CLAUDE.md`)은 **`**...**` 내부에 구두점(괄호, 따옴표, 콜론 등)이 포함된 경우**에만 닫는 `**` 뒤 공백을 요구한다. 단순한 `**Paperclip**은`, `**control plane**이다`는 내부 구두점이 없어 CommonMark right-flanking 규칙상 깨지지 않는다.
- 검증한 위반 사례:
  - `**그림 1-A**가` (39줄, A8에서 별도 처리)
  - `**자동 재할당·자동 치유 안 함**` (72줄, `·` 포함) — 깨질 수 있어 좁혀서 공백 추가
- 그 외 Codex가 든 위치들은 규칙 적용 대상이 아니므로 REJECT. 일괄 공백 삽입은 본문 가독성만 해친다.

## DEFER (0개)

없음.

## 누적 LOC 추적

- 챕터 00 수정 예상 변경 라인: \~20\~30줄 (8개 항목 × 평균 2\~4줄)
- 11챕터 총 누적 예상: \~250\~400 LOC (200 LOC mandatory checkpoint 초과 예상). 챕터 03 또는 04 즈음 사용자에게 보고 예정.
