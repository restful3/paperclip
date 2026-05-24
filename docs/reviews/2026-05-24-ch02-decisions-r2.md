# ch02 Round 2 — Claude Decisions

_2026-05-24, codex-a / 1 정합성 finding_

## ACCEPT

### F1 — line 11: 12\~13개 → 12개 이상 (그림 캡션과 정합)
- **AS-IS**: `12\~13개 테이블만 추려 그림 4로 묶었다`
- **TO-BE**: `12개 이상 테이블을 추려 그림 4로 묶었다`
- **근거**: 그림 캡션은 이미 "12개 이상 테이블". 실제 SVG 에 14~15개 박스가 있어 "12~13" 으로 고정 어려움.

## 변경 적용 파일
- `docs/report/02-data-model.md` (line 11, 1곳)

## Round 3 송부 의도
FINAL_APPROVAL 기대.
