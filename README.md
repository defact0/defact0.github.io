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

# 테마 다운로드
git clone https://github.com/defact0/defact0.github.io.git

# 로컬에서 실행
bundle install
jekyll serve
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


# Reference
- https://github.com/mmistakes/minimal-mistakes/discussions/1352
- https://noonnu.cc/font_page/40
- https://amsomad.github.io/%EC%9C%A0%ED%8B%B8%EB%A6%AC%ED%8B%B0/imal_Mistakes_font_update/
- https://djccnt15.github.io/webdev/blog_customizing/
- https://danggai.github.io/github.io/Github.io-%EA%B2%8C%EC%8B%9C%EA%B8%80-%EB%B0%91%EC%A4%84-%EC%A0%9C%EA%B1%B0,-%EB%B3%B8%EB%AC%B8-%EA%B8%80%EC%9E%90-%ED%81%AC%EA%B8%B0-%EC%88%98%EC%A0%95/