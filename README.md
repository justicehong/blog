# Justice Hong · Dev Notes

서버·백엔드 개발자 홍정의의 기술 블로그입니다.
눈여겨본 오픈소스와 도구, 개발하면서 정리한 내용을 기록합니다.

- **Blog:** https://justicehong.github.io/blog/

## 구성

| 경로 | 내용 |
|---|---|
| `_posts/` | 게시물 (마크다운). 파일 하나가 글 하나 |
| `_layouts/` | 공통 틀(`default.html`)과 글 상세(`post.html`) |
| `index.html` | 글 목록 (최신순) |
| `assets/css/main.css` | 전체 스타일. 색·글꼴·간격은 파일 맨 위 CSS 변수(`:root`)에서 관리 |
| `_config.yml` | 사이트 제목·주소·작성자 정보 |

## 디자인

Claude 디자인 문서(DESIGN.md)의 토큰을 따릅니다.

- 크림 캔버스(`#faf9f5`)에 따뜻한 잉크색 글자, 코랄은 본문 링크와 주요 버튼에만 사용
- 제목은 세리프(굵기 400), 본문은 산세리프, 코드는 JetBrains Mono
- 그림자 대신 면 색으로 구분: 크림 → 크림 카드 → 다크 코드 창 → 다크 푸터
- Copernicus / StyreneB는 공개 웹폰트가 아니어서 Source Serif 4 + Noto Serif KR, Inter + Noto Sans KR로 대체

## 글 쓰는 방법

1. `_posts/` 아래에 `YYYY-MM-DD-영문-제목.md` 파일을 만듭니다.
   파일명의 영문 제목이 글 주소가 됩니다. 예: `2026-12-01-my-post.md` → `/blog/posts/my-post/`
2. 파일 맨 위에 아래 머리말을 넣고, 그 아래에 마크다운으로 본문을 씁니다.

   ```markdown
   ---
   layout: post
   title: "글 제목"
   description: "목록과 검색 결과에 보이는 한두 문장 요약"
   date: 2026-12-01 09:00:00 +0900
   tags: [태그1, 태그2]
   ---

   본문...
   ```

3. `main` 브랜치에 push하면 GitHub Pages가 자동으로 빌드해 1~2분 내 반영됩니다.

`date`가 미래 시각이면 그 시각이 지나고 다시 빌드될 때까지 글이 보이지 않습니다.
본문에서 지원하는 요소: 제목(`##`, `###`), 목록, 표, 코드 블록, 인용문(`>`), 링크, 이미지.

## 배포

GitHub Pages — `main` 브랜치 루트를 Jekyll로 빌드합니다.
(Settings › Pages › Build and deployment › Source: **Deploy from a branch**, Branch: `main` / `/ (root)`)

별도 빌드 설정이나 Actions 워크플로는 필요 없습니다.
레포 이름을 바꾸면 `_config.yml`의 `baseurl`도 함께 바꿔야 합니다.

## 로컬 미리보기 (선택)

```bash
bundle install
bundle exec jekyll serve
# http://localhost:4000/blog/
```

Ruby가 없다면 push 후 실제 사이트에서 확인해도 됩니다.

## 라이선스

- 콘텐츠(글·이미지): © 홍정의. 무단 전재를 금합니다.
