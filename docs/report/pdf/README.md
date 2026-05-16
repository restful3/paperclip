# PDF 산출물

이 폴더는 `docs/report/` 의 11개 챕터(00\~10) 를 *PDF 전용 표지/목차* 와 함께 한 권으로 빌드한 결과물이다.

## 산출물

- `paperclip-learning-report.pdf` (\~6.7 MB, 77 페이지) — 인쇄/배포용 PDF
  - **풀페이지 표지** — A4 한 페이지를 가득 채우는 커버
  - **목차 페이지** — 11개 챕터 한 줄 요약 + 그림·표 인덱스 + 시각 스타일 규칙
  - **PDF outline (북마크)** — 헤딩(h1\~h6) 기반 자동 생성. PDF 뷰어 북마크 패널에서 챕터/절 단위 점프 가능
  - **그림·표 통일 캡션** — 모든 그림/표 위에 `**그림 N. 〈제목〉**` / `**표 N. 〈제목〉**` 형식의 본문 라벨
- `paperclip-learning-report.html` — 같은 본문의 HTML 미리보기
- `figs/` — PDF/HTML 이 참조하는 SVG/이미지 자산 (28개)

## 빌드 환경

- 빌더: [slideflow](https://github.com/restful3/slideflow) (`/home/restful3/workspace/slideflow/`)
- 테마: `research` (네이비 + 산세리프, 학술 리서치 톤)
- 브랜드: `ai-odyssey` (Independent Research 페르소나)
- 입력 stem: `docs/report/_pdf-build/`
  - **`00_index.md`** — *PDF 전용 표지/목차 페이지* (`docs/report/README.md` 와 별개의 풍부한 frontmatter + inline `<style>` 로 풀페이지 표지·목차 스타일을 정의)
  - **`00_overview.md` \~ `10_research-map.md`** — `docs/report/00-*\~10-*.md` 챕터 본문을 알파벳순 결합 가능하게 `NN_*.md` 로 재명명하고 `assets/` 경로를 `figs/` 로 일괄 치환한 사본
  - **`figs/`** — `docs/report/assets/` 의 사본 (SVG 10 + 외부 이미지 17 + MANIFEST)

## 재빌드 한 줄

```bash
cd /home/restful3/workspace/slideflow
uv run python build.py \
    /home/restful3/workspace/paperclip/docs/report/_pdf-build \
    --theme research --brand ai-odyssey --pdf

# 산출물 → /home/restful3/workspace/slideflow/dist/report/_pdf-build--research--ai-odyssey.{html,pdf}
# 그 뒤 docs/report/pdf/ 로 복사
```

> slideflow `build.py` 가 Chrome `--generate-pdf-document-outline` 플래그를 자동 부착하므로, 별도 옵션 없이 PDF 북마크가 자동 생성된다. 헤딩 구조가 본문에 잘 잡혀 있으면 그대로 동작.

## 본문을 수정한 경우

`docs/report/` 의 챕터 본문(00\~10) 을 수정한 뒤 PDF 를 다시 만들려면 `_pdf-build/` 의 챕터 사본을 갱신해야 한다. **`00_index.md` 는 PDF 전용 표지/목차이므로 덮어쓰지 말 것.**

```bash
cd /home/restful3/workspace/paperclip/docs/report/_pdf-build

# 11개 챕터 본문만 갱신 (00_index.md 는 보존)
for f in ../00-overview.md ../01-architecture.md ../02-data-model.md \
         ../03-runtime-execution.md ../04-adapters-and-skills.md ../05-server-api.md \
         ../06-ui-and-board.md ../07-governance-cost.md ../08-usage.md \
         ../09-pros-cons.md ../10-research-map.md; do
  base=$(basename "$f")
  out=$(echo "$base" | sed 's/^\([0-9][0-9]\)-/\1_/')
  sed 's|assets/|figs/|g' "$f" > "$out"
done

# figs 도 다시 동기화 (그림 SVG 가 바뀌었을 수 있음)
rm -rf figs && cp -r ../assets ./figs

# 그리고 위 "재빌드 한 줄" 실행
```

## 표지/목차를 수정하려면

`_pdf-build/00_index.md` 직접 편집:
- **표지 메타** — frontmatter 의 `eyebrow` / `title` / `subtitle` / `author` / `date` / `version` / `publisher` / `period`
- **풀페이지 표지 CSS** — 본문 상단 `<style>` 블록의 `.cover { ... }` 규칙
- **목차** — `<ul class="toc-list">` 안의 `<li>` 항목 (챕터 한 줄 요약)

> `_pdf-build/` 는 빌드 입력용 임시 디렉터리이며 학습 자료가 아니다. `docs/report/README.md` §1 의 *읽는 순서* 에는 포함되지 않는다.
