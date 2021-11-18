# Vue.js 란?

> **progressive framework** for building user interfaces

- 2014 년 11월 9일에 Vue.js v0.11 Release 후 현재 v2.6.10 까지 Release 됨  
  [vue - Roadmap | github](https://github.com/vuejs/vue/projects/6)
- Google Creative Lab에서 일하던 [EVAN YOU](https://blog.evanyou.me/) 가 개발
- 현재 가장 주목받고 있는 Frontend Framework 중 하나  
  <img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_1/1-1.png" width="400">

  <sup>이미지 출처 :
  [risingstars.js.org](https://risingstars.js.org/2018/en/)
  </sup>

## npm-stat 확인

[npm-stat.com](https://npm-stat.com/charts.html?package=react&package=vue)

<img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_1/1-2.png" width="400">

> 사용량으로 보면 `React`가 월등히 앞서고 있음 <g-emoji>🙄</g-emoji>  
> but, 2~3년 전과 비교해 본다면 그 격차가 매우 줄었음

## Developer Survey Results 2019

> **React.js and Vue.js are both the most loved and most wanted web frameworks by developers, while Drupal and jQuery are most dreaded.**

<img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_1/1-3.png" width="600">

<sup>이미지 출처 :
[stackoverflow-Developer Survey Results 2019](https://insights.stackoverflow.com/survey/2019#technology-_-most-loved-dreaded-and-wanted-web-frameworks)
</sup>

> `Vue.js`는 매우 사랑받는 Web Framework 중 하나이고 `Jquery`는 기피하는 현상을 볼 수 있음

# Vue.js 의 장점

## 가볍고 유연함

### **ViewModel layer에 초점을 맞추어 핵심 라이브러리가 개발됨**

<img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_1/2-1.png" width="400">

<sup>이미지 출처 :
[Vue.js 입문자를 위한 공개 세미나](https://www.slideshare.net/GihyoJoshuaJang/do-it-vuejs-88453012?from_action=save)
</sup>

- Lean & Small (**16KB** minified and gzipped)
- 실제 개발 시에는 Vue.js 관련 라이브러리 사용이 필수

### **전체 아키텍처를 새롭게 구성할 필요가 없음** (점진적 프레임워크)

- 기존 앱의 일부 화면에 적용해본 후 전체 적용을 검토해볼 수 있음

### **SPA(Single Page Application)를 매우 잘 지원함**

- Router 기능 지원 (Client Side Routing)
- Dynamic UI 의 웹 페이지를 만들 때 유리

## 진입 장벽이 (상대적으로) 낮음

- (상대적으로) 간단하고 쉬운 코드
- 필요 기술 스택

  <img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_1/2-2.png" width="400">

  <sup>이미지 출처 :
  [Vue.js 입문자를 위한 공개 세미나](https://www.slideshare.net/GihyoJoshuaJang/do-it-vuejs-88453012?from_action=save)
  </sup>

- `<script>` 태그를 활용하여 CDN 주소를 추가한 후 vue.js 디렉티브 몇 가지를 익히면 바로 프로그래밍 가능
- 문서가 매우 잘 정리되어 있고, 한글 번역도 매우 훌륭함
- ~~개떡~~같이 만들어도 동작은 잘 하는편 (React같이 엄격하지 않음, Magic이 있음)

> 공식문서 <g-emoji>👉</g-emoji>[시작하기 - Vue.js](https://kr.vuejs.org/v2/guide/)  
> 추천 영상 <g-emoji>👉</g-emoji> [Learn Vue.js with our Courses - Vue Mastery](https://www.vuemastery.com/courses/) (일부 유료)

## 리액트와 앵귤러의 장점을 포함하면서 Jquery를 대체 가능한 프레임워크

### 2-Way Binding (Angular)

<img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_1/2-3.png" width="300">
<img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_1/2-4.gif" width="300">

### Virtual DOM (React)

<img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_1/2-5.png" width="300">

<sup>이미지 출처 :
[React and the Virtual DOM](https://www.youtube.com/watch?v=BYbgopx44vo)
</sup>

> Virtual DOM은 `Vue.js 주요컨셉` 설명할 때 자세히 언급할 예정입니다.

### Jquery로 했던 작업들은 모두 대체 가능

jQuery는 DOM을 직접 다루는 기능이 너무 많기 때문에 안티패턴의 위험이 높음

> 그럴수 없겠지만 **jQuery는 이제 그만 쓰는게 좋습니다 ㅠ.ㅠ**

| #           | Jquery                  | Vue.js             |
| ----------- | ----------------------- | ------------------ |
| 구분        | 자바스크립트 라이브러리 | Frontend Framework |
| 렌더링 방식 | DOM에 직접 접근         | Virtual DOM을 활용 |
| 렌더링 속도 | 16,862 ms               | 2,985 ms           |
| 메모리 사용 | 62 MB                   | 171 MB             |

> 위 표에서 렌더링 속도와 메모리 사용 값은 특정 상황을 가정하여 여러 Front-End Framework의 속도 및 성능을 테스트 한 결과값을 표기하였습니다.  
> ([출처 : Rendering Speed & Performance challenge with Famous Front-End Framework](https://medium.com/thothzocial-engineering/rendering-speed-performance-challenge-with-famous-front-end-framework-196c876a68af) )

# Vue.js 주요컨셉

## Inspired by MVVM Pattern

### View

- 사용자 UI (HTML + CSS)
- 주요 기능
  - 사용자에게 데이터를 Presentation
  - 사용자로부터 Input을 받음

### <span style="color:green;font-weight:bold">ViewModel</span>

- View를 추상화한 개념
- View의 실제 논리 및 데이터 흐름 담당
- 주요 기능
  - View와 ViewModel의 상태를 동기화 하기 위해 `DataBinding` 기능 제공  
    <g-emoji>👉</g-emoji> ViewModel이 바뀌면 View도 변경됨
  - 사용자가 View에 전달한 Input을 실행하기 위해 `Command` 들을 제공

### Model

- only 데이터만 정의함
- View에서 어떻게 보이는지 혹은 ViewModel에 어떤 비지니스 로직이 있는지 등을 신경 쓸 필요가 <g-emoji>❌</g-emoji>
- 특정 프레임워크나 기술에 종속될 필요 <g-emoji>❌</g-emoji>

<img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_1/3-1.png" width="500">

> **MVVM Pattern을 사용하는 이유**  
> 애플리케이션의 로직(ViewModel)과 사용자 UI(View)를 분리한 후 서로를 느슨하게 결합시키기 위함  
> 이렇게 하면 테스트하기가 좋고, 유지 보수 측면에서도 매우 유리함

## Single File Component

- 하나의 `.vue`파일에서 HTML/CSS/JavaScript를 모두 기술하게 함
  ```js
  <!-- my-component.vue -->
  <template>
    <div>이 곳은 사전에 컴파일 됨</div>
  </template>
  <script src="./my-component.js"></script>
  <style src="./my-component.css"></style>
  ```
- **관심사의 분리**
  - 타입별 파일 분리하는 것이 아니라 느슨하게 결함 된 컴포넌트로 나누어 구성
  - Single File Component 컨셉을 지키게 되면 컴포넌트의 응집력과 유지 보수와 재사용성이 높아짐

> 현대적인 UI 개발에서 코드베이스를 서로 얽혀있는 세 개의 거대한 레이어로 나누는 대신,  
> 느슨하게 결합 된 컴포넌트로 나누고 구성하는 것이 더 중요

> `.vue` 파일은 브라우저가 인식할 수 없기 때문에 별도의 작업이 필요함  
> (자세한 내용은 `lecture_4` 에서 다룰 예정입니다.)

## Virtual DOM

**효율적인 DOM 조작을 위해 사용**

### 브라우저의 Workflow

<img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_1/5.png" width="600">

<sup>이미지 출처 :
[ReactJS의 Virtual DOM과 Repaint, Reflow](http://blog.drakejin.me/React-VirtualDOM-And-Repaint-Reflow/)
</sup>

- 브라우저가 전달받은 HTML을 파싱하여 DOM 트리를 만듬 (HTML Element <g-emoji>👉</g-emoji> DOM Node)
- CSS 파일과 inline style 파싱
- Attachment : 각 DOM 노드들의 스타일 처리하는 과정  
  (DOM Tree + Style Rules = Render Tree)
- Layout : 각 노드들의 스크린의 좌표가 주어짐
- Painting : 렌더링 된 요소들에 색을 입히는 과정

<g-emoji>😫</g-emoji> **동적인 UI를 만들 경우 DOM과 CSS 조작이 매우 빈번하게 일어나게 되고,**  
**이는 리소스를 상대적으로 많이 차지하는 Reflow(Layout)와 Repaint 작업을 계속 발생시킴**

> 예를 들어 30개의 노드를 하나 하나 수정하면, 그 뜻은 30번의 (잠재적인) 레이아웃 재계산과 30번의 (잠재적인) 리렌더링을 초래한다는 뜻

<g-emoji>😎</g-emoji> **이런 비효율을 개선하기 위해 Virtual DOM이 등장**

<img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_1/6.png" width="500">

<sup>이미지 출처:
[Virtual Dom](https://medium.com/naukri-engineering/naukriengineering-virtual-dom-fa8019c626b)</sup>

- Virtual DOM은 실제 브라우저에 렌더링 하지 않고 메모리 상에만 존재하는 개념이므로 연산 비용이 매우 적음
- Real DOM 대신 DOM 트리의 변화를 체크하고 **모든 변화를 하나로 묶어서 던져주면 연산의 횟수를 줄일 수 있음**
  (~~연산의 규모는 커질 수 있지만~~)

# 개발환경 설정

## Node.js 설치

- [Node.js 공식 홈페이지](https://nodejs.org/ko/)에서 LTS 버전 다운로드 및 설치  
  <img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_1/7.png" width="400">

- node 설치 확인
  ```bash
  $> node -v
  v10.16.3
  ```
- npm 최신버전 업그레이드 및 확인
  ```bash
  $> npm install -g npm
  $> npm -v
  6.9.0
  ```
  > Mac OS 사용시 명령어 앞에 `sudo`를 꼭 붙여야 함. 안 붙이면 permission denied 에러 발생 ㄷㄷㄷ

## Visual Studio Code 설치

- [Visual Studio Code 공식 홈페이지](https://code.visualstudio.com/)에서 다운로드 및 설치
- Vue.js 개발 관련 플러그인  
  <img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_1/8.png" width="400">

## Vue.js devtools 설치

- 크롬 브라우저에서 웹스토어 검색 또는 아래 사이트로 이동
  https://chrome.google.com/webstore/category/extensions
- `Vue.js devtools` 검색 후 `Chrome에 추가` 버튼 클릭
  <img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_1/9.png" width="500">

- url 입력 창 옆에 아이콘 추가됐는지 확인
- [chrome://extensions/](chrome://extensions/) 에서 Vue.js devtools 세부정보 중 `확장 파일 URL에 대한 액세스 허용` 설정

## Vue-cli 설치

- 커맨드 라인 인터페이스 기반의 스캐폴딩 도구
- 설치 및 확인
  ```bash
  ## yarn 패키지 매니저도 같이 설치
  $> npm install -g yarn @vue/cli
  $> vue -V
  3.9.3
  ```

> **Mac에서 vue-cli 설치 시 오류가 날 수 있음**  
> `sudo npm install -g @vue/cli --unsafe-perm` 로 설치하면 해결됨  
> 출처 : [Errors during install of vue-cli on Mac OSX](https://github.com/vuejs/vue-cli/issues/3402)

# 첫번째 Vue.js Application

> 아주 간단한 Application 이기 때문에 [CodePen](https://codepen.io/pen/)에서 실습

- html
  ```html
  <!DOCTYPE html>
  <html>
    <head>
      <meta charset="utf-8" />
      <title>Hello Vue.js</title>
      <script src="https://unpkg.com/vue/dist/vue.js"></script>
    </head>
    <body>
      <div id="app">
        <h2>{{message}}</h2>
      </div>
    </body>
  </html>
  ```
- javascript
  ```javascript
  var model = {
    message: "Hello Vue.js",
  };
  var app = new Vue({
    el: "#app",
    data: model,
  });
  ```
- 모두 입력 후 `Ctrl` + `Enter`

# 첫번째 Vue.js Application 분석

- Vue 객체 생성 시 지정된 옵션에 의해 **HTML요소**와 **데이터** 참조
- 선언적으로 HTML DOM에 데이터를 렌더링 하기 위해 콧수염 표현식 사용
  <img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_1/10.png" width="600">

## 참고자료

[Vue.js 퀵 스타트](http://www.yes24.com/Product/Goods/45091747)  
[stepanowon/vuejs_book_2nd: Vue.js QuickStart 2판](https://github.com/stepanowon/vuejs_book_2nd)  
[Our Courses | Vue Mastery](https://www.vuemastery.com/courses/)  
[React 인가 Vue 인가?](https://joshua1988.github.io/web_dev/vue-or-react/)  
[Vue.js 입문자를 위한 공개 세미나](https://www.slideshare.net/GihyoJoshuaJang/do-it-vuejs-88453012)  
[[Vuetorials] 2. 전반적인 concept](https://jaeyeophan.github.io/2018/10/21/Vuetorials-2-Vue-concept/)  
[[Vue.JS] 반응형 시스템](https://beomy.tistory.com/66)  
[이제와서 JQUERY를 쓰면 안되는 이유, 혹은 JQUERY와 웹개발의 역사](https://www.tokyobranch.net/archives/6598)  
[MVVM 아키텍처 패턴 | Just hack'em](https://justhackem.wordpress.com/2017/03/05/mvvm-architectural-pattern/?source=post_page-----bb7576e23c65----------------------)  
[너무 쉬운 Vue.js](https://brunch.co.kr/@skykamja24/254)
