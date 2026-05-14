---
format: report
target: A4
topic: Paperclip 학습용 분석 리포트
---

# Paperclip 분석 리포트 — 인덱스

이 폴더는 Paperclip(https://github.com/paperclipai/paperclip) 코드베이스를 구조적으로 이해하기 위한 **분석 리포트**다. 독자가 제품의 문제 정의, 런타임 모델, 데이터 모델, API, UI, 거버넌스, 비용 통제까지 한 흐름으로 파악할 수 있도록 구성했다.

리포트는 **11개 장과 시각 자료**로 구성된다. 각 장은 하나의 분석 주제를 다루며, 그림과 표는 본문 번호로 인용된다.

## 1. 읽는 순서

| 단계 | 파일 | 무엇을 얻는가 |
|---|---|---|
| 1 | [00-overview.md](00-overview.md) | Paperclip의 정체성·문제 정의·"control plane vs execution plane" 분리 |
| 2 | [01-architecture.md](01-architecture.md) | 모노레포 패키지 지도 · 의존 관계 |
| 3 | [02-data-model.md](02-data-model.md) | Drizzle 스키마 80+ 테이블 핵심 ER |
| 4 | [03-runtime-execution.md](03-runtime-execution.md) | Heartbeat · atomic checkout · watchdog · stranded recovery |
| 5 | [04-adapters-and-skills.md](04-adapters-and-skills.md) | built-in 어댑터 + skills + Paperclip SKILL.md |
| 6 | [05-server-api.md](05-server-api.md) | Express 라우트 · 서비스 · 인증 · WebSocket 라이브 이벤트 |
| 7 | [06-ui-and-board.md](06-ui-and-board.md) | React 보드 UI · 페이지 · 플러그인 |
| 8 | [07-governance-cost.md](07-governance-cost.md) | 보드 거버넌스 · 예산 · 승인 · 활동 로그 |
| 9 | [08-usage.md](08-usage.md) | 설치 · 개발 · 회사 만들기 · 운영 |
| 10 | [09-pros-cons.md](09-pros-cons.md) | 강점·약점·트레이드오프·성숙도 |
| 11 | [10-research-map.md](10-research-map.md) | 리서치 ↔ Paperclip 위계 지도 |

## 2. 보조 자료

- **그림 자산**: `assets/fig-01-bigpicture.svg` \~ `assets/fig-10-research-hierarchy.svg`. 챕터 본문에 그림 번호로 인용된다.
- **외부 이미지**: `assets/external/`가 있다면 인용된 원본 페이지에서 수집한 공식 다이어그램이다. `MANIFEST.md`가 출처를 기록한다.
- **리서치 원본**: 폴더 외부의 [`../research/01\~10-*.md`](../research/) 에 정리되어 있다. 각 파일은 `format: report` 프런트매터를 갖고 있으며, 문장 단위 출처 URL을 포함한다.

## 3. 시작 시 체크 사항

```bash
# 저장소 루트로 이동
cd /home/restful3/workspace/paperclip

# 사전 준비: Node 20+, pnpm 9+
node --version && pnpm --version

# 의존성 설치
pnpm install

# 임베디드 PostgreSQL 모드로 즉시 dev 시작 (DATABASE_URL 미설정)
pnpm dev
# → http://localhost:3100  (UI · API 동일 origin)
```

## 4. 시각 스타일

- 잉크 색 `#1a1a2e`, 액센트 레드 `#c84a3a`, 네이비 `#0f2c59`. 회색 톤 `#5b5b62`.
- 표·그림 번호는 본문에서 `**그림 N. 〈제목〉**` 와 `**표 N. 〈제목〉**` 형태로 부여한다. 외부 이미지는 본문 캡션에 출처를 함께 표기한다.
- SVG 본체에는 제목/캡션을 넣지 않는다. 다이어그램 내부의 데이터 라벨(축, 박스 안의 짧은 설명)은 허용한다.
- 한국어 본문 + 영어 고유명사 병기. CJK bold 규칙(`**용어(English)**는`)을 따른다.

## 5. 라이선스 / 저작권 안내

이 리포트는 학습 목적의 2차 정리물이다. 인용한 외부 이미지는 모두 출처 페이지로 링크되며, 원 저작자의 권리를 그대로 둔다. 인용 부적절성이 발견되면 `assets/external/MANIFEST.md`의 해당 항목과 본문 인용을 제거하면 된다.
