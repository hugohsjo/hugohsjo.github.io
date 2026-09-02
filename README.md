# hugojo.com 블로그 저장소 가이드

휴고조 Hugo Jo 블로그의 소스 저장소다. 이 문서 하나로 글 작성부터 발행까지 혼자 할 수 있도록 구조, 역할, 커스터마이징 현황을 전부 기록한다.

- 로컬 경로: `~/Library/CloudStorage/Dropbox/Studio/hugojo.com`
- 원격: github.com/hugohsjo/hugohsjo.github.io
- 공개 주소: https://hugohsjo.github.io (도메인 연결 후 https://hugojo.com)
- 스택: Hugo(정적 생성기) + PaperMod 테마 + GitHub Actions + GitHub Pages

## 1. 발행 흐름 (핵심 원리)

```
마크다운 작성 → git commit → git push → GitHub Actions가 자동 빌드 → 약 1분 뒤 사이트 반영
```

로컬에서 빌드할 필요 없다. push가 곧 발행이다.

## 2. 디렉토리와 파일

| 경로 | 역할 | 상태 |
|---|---|---|
| `content/` | **글 원본 전부. 매일 만지는 유일한 곳** | 커스텀 |
| `content/connect/` | Connect 축 글 (Amazon Connect·AICC 구축기) | 커스텀 (빈 섹션) |
| `content/build/` | Build 축 글 (앱·웹서비스 개발기) | 커스텀 (빈 섹션) |
| `content/life/` | Life 축 글 (만드는 사람의 생각) | 커스텀 |
| `content/about.md` | About 페이지 (/about/) | 커스텀 |
| `content/search.md` | 검색 페이지 (/search/) | 커스텀 |
| `hugo.toml` | 사이트 전체 설정. 제목·메뉴·소셜 링크 등 | 커스텀 (아래 3번) |
| `archetypes/default.md` | `hugo new`로 새 글을 만들 때의 front matter 템플릿 | 커스텀 |
| `static/` | 빌드 시 그대로 복사되는 정적 파일 (파비콘, 이미지) | 커스텀 (파비콘 2개) |
| `themes/PaperMod/` | 테마 (git 서브모듈). **직접 수정 금지** | 기본 설치 |
| `.github/workflows/hugo.yaml` | push 시 자동 빌드·배포하는 GitHub Actions 정의 | 기본 설치 |
| `.gitmodules` | 테마 서브모듈 등록 정보 | 기본 설치 |
| `.gitignore` | 빌드 산출물(`public/`, `resources/`) git 제외 | 커스텀 |
| `public/`, `resources/` | 로컬 빌드 산출물. git에 올라가지 않음, 지워도 됨 | 자동 생성 |
| `.hugo_build.lock` | Hugo 빌드 잠금 파일. 신경 쓸 필요 없음 | 자동 생성 |

테마 수정 금지의 이유: 테마는 서브모듈이라 수정하면 업데이트가 막힌다. 꾸밈 변경이 필요하면 `hugo.toml`의 params 또는 루트에 `assets/css/extended/` 폴더를 만들어 CSS만 얹는 방식을 쓴다(PaperMod 공식 확장 방법).

## 3. hugo.toml 커스터마이징 현황

기본 골격에서 변경·추가된 항목 전부:

- `title = "휴고조 Hugo Jo"`, `description` = 확정 태그라인
- `languageCode = "ko-kr"`, `timeZone = "Asia/Seoul"`, `enableRobotsTXT = true`
- `[outputs] home = ["HTML", "RSS", "JSON"]` : JSON은 검색 페이지가 사용. 지우면 검색이 죽는다
- `[params.homeInfoParams]` : 홈 화면의 브랜드 문장
- `[[params.socialIcons]]` 7개 : YouTube·Instagram·Threads·X·GitHub·LinkedIn·RSS
- `[params.assets]` : 파비콘 연결 (`static/favicon-32.png`, `static/apple-touch-icon.png`)
- `[menu]` : Connect / Build / Life / About / 검색, weight 순서로 정렬
- 표시 옵션 : 읽기 시간, 브레드크럼, 이전·다음 글, 코드 복사 버튼 켬

## 4. 글 쓰는 법

### 새 글 만들기

저장소 루트에서:

```bash
hugo new build/my-post-slug.md
```

- `build/` 자리에 축을 넣는다: `connect/`, `build/`, `life/`
- **파일명이 곧 URL**이다. 영문 소문자와 하이픈만 사용 (예: `connect/first-call-routing.md` → `/connect/first-call-routing/`)
- 생성된 파일의 front matter를 채운다:

```yaml
---
title: "Amazon Connect 구축, 국내 리전에서 처음 결정할 것들"
date: 2026-09-07T20:00:00+09:00
draft: true
categories: ["Connect"]
tags: ["AmazonConnect", "AICC"]
summary: "목록과 검색 결과에 노출되는 한두 문장 요약"
---
```

- `categories`는 축과 동일하게 하나만: `Connect` / `Build` / `Life`
- 본문은 그 아래에 마크다운으로 작성

### 이미지 넣기

글 하나에 이미지가 여러 장이면 페이지 번들 방식을 쓴다. 파일 대신 폴더를 만들고 글은 `index.md`로:

```
content/build/gageboo-launch/
├── index.md
├── screenshot-1.png
└── screenshot-2.png
```

본문에서는 `![설명](screenshot-1.png)`처럼 파일명만으로 참조한다.

### 미리보기

```bash
hugo server -D
```

브라우저에서 http://localhost:1313 접속. `-D`는 draft 글도 보여준다. 저장할 때마다 자동 새로고침된다.

### 발행

1. front matter의 `draft: true` 줄을 지우거나 `false`로
2. 커밋하고 푸시:

```bash
git add -A && git commit -m "post: 글 제목" && git push
```

약 1분 뒤 사이트에 반영된다. 배포 상태 확인은 `gh run list --limit 1`.

## 5. 글쓰기 규칙 (전략 문서 요약)

- 제목은 검색어형: 타깃 검색어로 시작한다. 브랜드 문장은 블로그명이 담당한다
- 글 구조: 도입 2~3줄(키워드 자연스럽게) → 과정·코드·스크린샷 → 유튜브 영상 임베드 → 다음 단계 예고와 구독 한 줄
- 축마다 허브(총정리) 글 1편을 만들고 모든 글과 상호 내부 링크
- 영상 임베드: `{{</* youtube 영상ID */>}}` 숏코드 사용

## 6. 도메인 전환 시 바꿀 것 (hugojo.com 연결 시점)

1. `hugo.toml`의 `baseURL`을 `https://hugojo.com/`으로
2. `static/CNAME` 파일 생성, 내용은 `hugojo.com` 한 줄
3. GitHub 저장소 Settings → Pages → Custom domain에 hugojo.com 입력, Enforce HTTPS 체크
4. 등록기관 DNS: apex에 A 레코드 4개 (185.199.108.153 / 185.199.109.153 / 185.199.110.153 / 185.199.111.153)
