---
title: "업무 보조도구(맞춤법 검사)"
date: 2024-01-27
last_modified_at: 2024-01-27
categories:
  - Tools
tags:
  - Python
  - Selenium
tagline: "Python기반의 Flask 웹 프레임워크와 Selenium을 활용한..."
header:
  overlay_image: /images/overlay_image.jpg
  overlay_filter: 0.5 ## same as adding an opacity of 0.5 to a black
---

최근에 공지를 작성해야 하는 일이 생겼는데 맞춤법 검사기를 활용하여 내용을 수정하고 있다.  

"네이버 맞춤법 검사기"를 사용하다 보니 불편한 점이 생기기 시작했는데,  
매번 해당 기능을 활용하려면 사이트에 직접 찾아들어가서 맞춤법을 검사하고 있어야 했고,  
한번 검사할 때마다 300자 제한이 걸려 있어 300자가 넘으면 내용을 잘라서 검사해야 했다.

![](/images/2024-01-27-spell-check/naver-spellcheck.png)

업무에 필요한 보조도구를 만들고 있던 차라 해당 기능도 추가로 만들기 시작했다.

### 어떻게 만들까?

보조도구는 `Python`의 `Flask` 웹 프레임워크를 사용했고 `Bootstrap`을 활용하여 화면에 적용했다.  
이제 문제는 어떻게 맞춤법 검사 기능을 구현할 것인가?인데...  
관련 자료를 찾다보니 `py-hanspell`이라는 라이브러리가 검색이 되었다.

### py-hanspell
* py-hanspell은 네이버 맞춤법 검사기를 이용한 파이썬용 한글 맞춤법 검사 라이브러리이다.
* [https://github.com/ssut/py-hanspell](https://github.com/ssut/py-hanspell)


### py-hanspell 문제점

`py-hanspell`을 사용하려니 기능이 정상적으로 동작하지 않아 추가적으로 확인해 보았더니 아래와 같은 문제점이 발생했다.

![](/images/2024-01-27-spell-check/passportKey.png)

1. 맞춤법 검사 요청을 보낼 때마다 `passportKey`, `callback`값을 요구한다.
2. `passportKey`, `callback`값은 고정값이 아니다.

라이브러리 제작 당시는 `passportKey`, `callback`값을 요구하지 않았는데 어느 순간 부터 요구하기 시작한 것 같다.  
실행환경마다 값이 변경되는 것 같아 매번 유효한 값으로 변경해 주어야 하는 문제가 발생했다.

### 어떻게 해결하지?

결국 생각나는 것은 `Selenium`을 활용하여 구현하는 방법으로 진행하게 되었다.  
Selenium은 웹 애플리케이션 자동화 및 테스트를 위한 포터블 프레임워크이다.

구현 방식은 아래와 같다.

1. 검사할 문장을 한 줄씩 네이버 맞춤법 검사기로 요청을 보낸다.
2. 검사 결과 값을 저장한다.
3. 검사할 문장이 없으면 저장된 내용을 화면에 출력한다.

#### 구현 결과 데모(GIF)

![](/images/2024-01-27-spell-check/spell-check.gif)

* 한 줄씩 요청하고 받기 때문에 시간이 걸리는 편이다.

#### headless chrome 모드

* 백그라운드 프로세스(background process) 형태로 실행하는 것이라 이해하면 된다.

```python
chrome_options = Options()
chrome_options.add_argument('--headless') # ChromeOptions를 사용하여 headless 모드로 설정
driver = webdriver.Chrome(service=ChromeService(ChromeDriverManager().install()), options=chrome_options) # 크롬 드라이버 생성
```