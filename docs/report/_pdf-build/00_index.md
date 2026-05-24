---
format: report
target: A4
theme: research
brand: ai-odyssey
eyebrow: Independent Research Report · 독립 리서치
title: Paperclip 학습용 분석 리포트
subtitle: 오픈소스 자율 에이전트 control plane 의 11챕터 가이드
author: Claude Opus 4.7 (1M context)
date: 2026-05-24
version: Phase G+
publisher: AI Odyssey · Independent Research
period: "Phase A → G+"
topic: Paperclip 학습용 분석 리포트
abstract: |
  <strong>Paperclip</strong> 은 자율 에이전트들이 모인 <em>회사</em> 를 운영하는 오픈소스 <strong>control plane</strong> 이다. 보드 운영자는 승인·예산·일시정지로 거버넌스를 행사하고, 어댑터(외부 CLI · 웹훅 · 원격 게이트웨이) 가 실제 에이전트 런타임을 맡는다. 이 리포트는 <strong>11개 챕터</strong> 로 데이터 모델·런타임·어댑터·서버·UI·거버넌스를 따라가며 큰 그림을 한 권에 잡는다.
kpi_stats:
  - { value: "11", label: "Chapters" }
  - { value: "28", label: "Figures" }
  - { value: "19", label: "Tables" }
  - { value: "79", label: "Pages" }
toc_pills:
  - { num: "00", title: "Overview" }
  - { num: "01", title: "Architecture" }
  - { num: "02", title: "Data Model" }
  - { num: "03", title: "Runtime Execution" }
  - { num: "04", title: "Adapters & Skills" }
  - { num: "05", title: "Server & API" }
  - { num: "06", title: "UI & Board" }
  - { num: "07", title: "Governance & Cost" }
  - { num: "08", title: "Usage" }
  - { num: "09", title: "Pros & Cons" }
  - { num: "10", title: "Research Map" }
---

<style>
/* =============================================================
   Paperclip 리포트 — 본문 콘텐츠 보조 스타일
   표지 / 챕터 표지 / 거대 챕터 번호 본 레이아웃은 slideflow research 테마가 담당.
   여기에는 paperclip-only: ① 목차 페이지 강제 break · ② 목차 리스트 스타일 · ③ 챕터별 hint 카피만 남는다.
   ============================================================= */

/* 목차 페이지 — body 의 첫 H2(목차) 앞에서 페이지 break */
.body > h2:first-of-type {
  page-break-before: always;
  break-before: page;
  margin-top: 0;
}

/* 목차 리스트 (점프 가능한 챕터 인덱스) */
.toc-list {
  list-style: none;
  padding: 0;
  margin: 28px 0 32px;
  break-inside: avoid;
}
.toc-list li.toc-chapter {
  margin: 0;
  padding: 0;
  border-bottom: 1px solid var(--hairline);
}
.toc-list li.toc-chapter a {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  gap: 16px;
  padding: 14px 4px;
  text-decoration: none;
  color: inherit;
  font-weight: 600;
  font-size: 1.05rem;
  transition: background 0.15s;
}
.toc-list li.toc-chapter a:hover { background: var(--accent-soft); }
.toc-list .toc-num {
  font-family: var(--font-mono);
  color: var(--accent);
  letter-spacing: 0.08em;
  font-weight: 800;
  min-width: 64px;
  flex-shrink: 0;
  font-size: 1rem;
}
.toc-list .toc-title { flex: 1; color: var(--ink); }
.toc-list .toc-hint {
  color: var(--ink-soft);
  font-size: 0.82rem;
  text-align: right;
  font-style: italic;
  font-weight: 400;
  flex-shrink: 0;
  max-width: 38%;
}

/* 챕터별 한 줄 카피 — 테마의 .body > h1::after 에 챕터별로 주입.
   ::before 의 거대 챕터 번호는 테마의 CSS counter 가 자동 처리(00, 01, 02 …) 하므로
   여기서는 hint(::after) 만 주입한다. */
.body > h1:nth-of-type(1)::after   { content: "정체성 · 3-zone · 5대 원칙"; }
.body > h1:nth-of-type(2)::after   { content: "pnpm workspace · 8 패키지 · dev 명령"; }
.body > h1:nth-of-type(3)::after   { content: "핵심 ER · atomic checkout · 도메인 군집"; }
.body > h1:nth-of-type(4)::after   { content: "시퀀스 · 7-state · stranded recovery"; }
.body > h1:nth-of-type(5)::after   { content: "능력 매트릭스 · 4-surface · MCP"; }
.body > h1:nth-of-type(6)::after   { content: "routes · services · realtime · auth"; }
.body > h1:nth-of-type(7)::after   { content: "70+ 페이지 · TanStack Query · 디자인 가이드"; }
.body > h1:nth-of-type(8)::after   { content: "승인 · 예산 6단계 · 감사"; }
.body > h1:nth-of-type(9)::after   { content: "5분 시작 · CEO 첫 에이전트 · 운영 점검"; }
.body > h1:nth-of-type(10)::after  { content: "11 강점 · 10 약점 · 7 트레이드오프"; }
.body > h1:nth-of-type(11)::after  { content: "4-층 위계 · 외부 다이어그램 · 학습 단계"; }
</style>

## 목차 — Table of Contents

이 리포트는 **표지 → 목차 → 11개 챕터** 순으로 구성된다. 아래 항목을 클릭(또는 PDF 뷰어의 outline 패널) 하여 해당 챕터로 점프할 수 있다. 각 챕터 머리에는 본문 라벨이 붙은 **그림** 과 **표** 가 배치되어 있다.

<ul class="toc-list">
  <li class="toc-chapter"><a href="#chapter-00"><span class="toc-num">00</span><span class="toc-title">Overview — Paperclip 은 무엇인가</span><span class="toc-hint">정체성 · 3-zone · 5대 원칙</span></a></li>
  <li class="toc-chapter"><a href="#chapter-01"><span class="toc-num">01</span><span class="toc-title">Architecture — 모노레포 구조</span><span class="toc-hint">pnpm workspace · 8 패키지 · dev 명령</span></a></li>
  <li class="toc-chapter"><a href="#chapter-02"><span class="toc-num">02</span><span class="toc-title">Data Model — 75+ 테이블 한 번에 보기</span><span class="toc-hint">핵심 ER · atomic checkout · 도메인 군집</span></a></li>
  <li class="toc-chapter"><a href="#chapter-03"><span class="toc-num">03</span><span class="toc-title">Runtime Execution — Heartbeat · Checkout · Watchdog</span><span class="toc-hint">시퀀스 · 7-state · stranded recovery</span></a></li>
  <li class="toc-chapter"><a href="#chapter-04"><span class="toc-num">04</span><span class="toc-title">Adapters &amp; Skills — 8개 어댑터 + Paperclip Skill</span><span class="toc-hint">능력 매트릭스 · 4-surface · MCP</span></a></li>
  <li class="toc-chapter"><a href="#chapter-05"><span class="toc-num">05</span><span class="toc-title">Server &amp; API — Express · Routes · Services · WebSocket</span><span class="toc-hint">routes · services · realtime · auth</span></a></li>
  <li class="toc-chapter"><a href="#chapter-06"><span class="toc-num">06</span><span class="toc-title">UI &amp; Board — React · Vite · shadcn/ui</span><span class="toc-hint">70+ 페이지 · TanStack Query · 디자인 가이드</span></a></li>
  <li class="toc-chapter"><a href="#chapter-07"><span class="toc-num">07</span><span class="toc-title">Governance &amp; Cost — Board · Approvals · Budget · Activity Log</span><span class="toc-hint">승인 · 예산 6단계 · 감사</span></a></li>
  <li class="toc-chapter"><a href="#chapter-08"><span class="toc-num">08</span><span class="toc-title">Usage — 설치 · 회사 만들기 · 첫 운영</span><span class="toc-hint">5분 시작 · CEO 첫 에이전트 · 운영 점검</span></a></li>
  <li class="toc-chapter"><a href="#chapter-09"><span class="toc-num">09</span><span class="toc-title">Pros &amp; Cons — Paperclip 의 트레이드오프</span><span class="toc-hint">11 강점 · 10 약점 · 7 트레이드오프</span></a></li>
  <li class="toc-chapter"><a href="#chapter-10"><span class="toc-num">10</span><span class="toc-title">Research Map — 10편의 리서치가 Paperclip 의 어디를 받치는가</span><span class="toc-hint">4-층 위계 · 외부 다이어그램 · 학습 단계</span></a></li>
</ul>

## 그림과 표 인덱스

본문에는 **자체 SVG 10장** + **외부 인용 그림 17장** + **표 21개** 가 배치되어 있다. 모두 동일한 캡션 형식 — `**그림 N. 〈제목〉**` / `**표 N. 〈제목〉**` — 으로 본문 라벨이 붙는다.

| 종류 | 위치 | 라벨 |
|---|---|---|
| 자체 그림 | 00\~10 챕터 | 그림 1 / 1-A · 2 · 3 · 4 · 4-A · 4-B · 5 · 5-A · 6 · 6-1 · 7 · 4-1\~4 · 8 · 9 · 10 · 10-1\~9 |
| 표 | 00\~10 챕터 | 표 1\~3 (각 챕터별 1\~3개) |

그림과 표의 *읽는 법* 은 본문 단락에 함께 주석되어 있다 — 각 그림/표가 어느 방향으로 읽는지, 어느 디테일을 주목해야 하는지, 다른 챕터의 어느 부분과 연결되는지 1\~2문단 분량으로 보강했다.

## 시각 스타일 규칙

- **잉크 색** `#1a1a2e`, **액센트 레드** `#c84a3a`, **네이비** `#0f2c59`, **회색 톤** `#5b5b62`.
- 표·그림 번호는 본문에서 `**그림 N. 〈제목〉**` · `**표 N. 〈제목〉**` 형식으로 부여한다(외부 이미지는 alt-text 에 출처 동행).
- SVG 본체에는 제목/캡션을 넣지 않는다(다이어그램 내부 데이터 라벨은 허용).
- 한국어 본문 + 영어 고유명사 병기. CJK bold 규칙(`**용어(English)** 는`) 준수.

## 라이선스 / 저작권 안내

이 리포트는 학습 목적의 2차 정리물이다. 인용한 외부 이미지는 모두 출처 페이지로 링크되며 원 저작자의 권리를 그대로 둔다. 인용 부적절성이 발견되면 `figs/external/MANIFEST.md` 의 해당 항목과 본문 인용을 제거하면 된다.
