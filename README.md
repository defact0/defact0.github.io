접속 주소: https://defact0.github.io/

## 환경구성

- https://www.ruby-lang.org/en/downloads/
- https://mmistakes.github.io/minimal-mistakes/


### 명령어

```shell
# 초기 구성
ruby -v
gem install jekyll
gem install bundler
gem install rake
gem install tocbot

# 로컬에서 실행
bundle install
jekyll serve

# 초안 작성 포스트 같이 보기
# --draft 폴더안에 내용 추가로 보여짐
jekyll serve --draft 

# 파일 삭제이후 실행
git clean -d -f
```

### 메인페이지 수정
- `_layouts/home.html`

```
classes: wide   # 추가

<div class="archive__item-teaser"> # 수동으로 추가...
  <img src="/images/main-image.jpg" alt="">
</div>
```

### 폰트
- `assets/css/main.scss`
- `_sass/minimal-mistakes/_variables.scss`

### 폰트사이즈 수정
- `_sass/minimal-mistakes/_reset.scss`

### Github.io 게시글 밑줄 제거, 본문 글자 크기 수정
- ` _sass/minimal-mistakes/로 이동하여, _base.scss` 

### 색상 수정
- `_sass\minimal-mistakes\skins\_default.scss`

# Reference
- https://github.com/mmistakes/minimal-mistakes/discussions/1352
- https://noonnu.cc/font_page/40
- https://amsomad.github.io/
- https://djccnt15.github.io/webdev/blog_customizing/
- https://danggai.github.io/github.io/Github.io-%EA%B2%8C%EC%8B%9C%EA%B8%80-%EB%B0%91%EC%A4%84-%EC%A0%9C%EA%B1%B0,-%EB%B3%B8%EB%AC%B8-%EA%B8%80%EC%9E%90-%ED%81%AC%EA%B8%B0-%EC%88%98%EC%A0%95/