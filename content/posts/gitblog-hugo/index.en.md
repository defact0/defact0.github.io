---
title: Hugo기반의 Gitblog만들기
date: 2021-04-04
tags: [markdown, gitblog, hugo]
categories: [gitblog]
featuredImage: "thumnail.png"
featuredImagePreview: "thumnail.png"
---

기존에 `Hexo` 기반의 `Theme NexT`테마를 쓰고 있었으나, 애니메이션 효과 때문인지 전반적으로 웹 페이지 반응이 느린거 같아 다른 테마를 찾아 사용하려고 한다.

<!--more-->

이번에는 `Hugo` 기반의 `LoveIt`테마를 사용할 것이다.  
**Hugo**는 jekyll, hexo 등과 같이 웹사이트를 쉽게 만들 수 있게 해주는 `static site generator`중 하나이다  
- static site는 고정된 html을 그냥 뿌려주는 사이트이다.  
따라서 static site를 쓴다면 언제 들어간다고 해도 항상 같은 화면만 나온다.  
- Hugo는 **Go** 라는 언어로 만들어졌다.

---

## 필요한 소프트웨어

1. git
    - https://git-scm.com/downloads
1. hugo
    - https://github.com/gohugoio/hugo/releases
        - hugo_extended_0.82.0_Windows-64bit.zip
            - extended 버전을 받아야 테마수정이 가능하다.
1. hugo theme
    - https://github.com/dillonzq/LoveIt
1. Visual Studio Code
    - https://code.visualstudio.com/

---

## hugo 환경설정

작업환경은 `windows` 에서 진행 한다.

1. [Hugo releases](https://github.com/gohugoio/hugo/releases)에서 다운로드 한다.  
`hugo_0.82.0_Windows-64bit.zip`을 다운받았다.
2. 압축을 풀면 exe 파일과 텍스트 파일 두개가 나오는 데 필요한건 exe 파일이다.
3. `C:\Hugo\bin` 경로에 exe 파일을 위치 시키고, `시스템 환경 변수 편집 > 환경 변수 > Path` 에 등록하였다.

---

## 블로그 생성
명령어를 입력하여 블로그를 생성해야 한다.  
`git bash`를 사용하거나 Visual Studio Code프로그램의 `bash 터미널`에서 작업한다.

```shell
hugo new site defact0
cd defact0
git clone https://github.com/dillonzq/LoveIt.git themes/LoveIt
```

---


## 블로그 테마 설정
테마 설정은 아래 페이지를 참고 하면된다.
- https://hugoloveit.com/theme-documentation-basics/
- https://hugoloveit.com/theme-documentation-content/
- https://hugoloveit.com/theme-documentation-built-in-shortcodes/
- https://hugoloveit.com/theme-documentation-extended-shortcodes/

---

## 블로그 폰트 변경
아래 경로에 있는 `_variables.scss`파일을 수정한다.
`/themes/LoveIt/assets/css/_variables.scss`

**원본 내용**
```scss
$global-font-family: system-ui, -apple-system, BlinkMacSystemFont, PingFang SC, Microsoft YaHei UI, Segoe UI, Roboto, Oxygen, Ubuntu, Cantarell, Fira Sans, Droid Sans, Helvetica Neue, Helvetica, Arial, sans-serif !default;
```

**수정 내용**
```scss
@import url('https://fonts.googleapis.com/css2?family=Nanum+Gothic&family=Noto+Sans+KR:wght@300&display=swap');
$global-font-family: 'Noto Sans KR', system-ui, -apple-system, sans-serif !default;
$global-font-size: 16px;
$global-font-weight: 300;
$global-line-height: 1.6rem;
```

---

## Categoris 페이지 more 버튼 제거

`/themes/LoveIt/layouts/taxonomy/terms.html` 파일을 수정
아래와 같이 주석 처리를 한다.

```html
{{- /*
<span class="more-post">
<a href="{{ .RelPermalink }}" class="more-single-link">{{ T "more" }} >></a>
</span>
*/ -}}
```

---

## 블로그 로컬 테스트

아래 명령어를 작업 폴더에서 입력하고 오류가 없다면, 웹 브라우저를 통해 http://localhost:1313 으로 접속하여 확인해 볼 수 있다.

```shell
hugo server
```

---

## 배포파일(Static files) 생성

```shell
hugo
```

위와 같이 hugo만 입력하면 `public` 디렉토리로 배포 파일을 생성됩니다.

## git에 push 하기


```shell
cd public/
git init
git remote add origin https://github.com/defact0/defact0.github.io.git
git add *
git commit -m "publish"
git push -u origin +master
```