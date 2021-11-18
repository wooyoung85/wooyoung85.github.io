# Vue CLI

- Vue.js 개발 환경을 빠르고 쉽게 제공
- 공식 지원되는 도구
- `project scffoding` 제공 (<g-emoji>👉</g-emoji> 프로젝트 폴더 구조 및 라이브러리 설정)
- Vue CLI를 쓰지 않고 직접 설정도 가능하지만 추천 <g-emoji>❌</g-emoji>

> ~~현재 `v4.0.0-rc.7` 까지 Pre-release 된 상태이지만 `3.x` version을 사용할 예정입니다.~~

> 게으른 탓인지 글을 쓰는 중에 `4.x` version이 공식 릴리즈 되었습니다 <g-emoji>😅</g-emoji>  
> `4.x` version 을 사용해야 할 것 같습니다 ㅎ

# Vue CLI 구성요소

## CLI

- `vue` 명령어 제공

  | command                                       | desc                                                                        |
  | --------------------------------------------- | --------------------------------------------------------------------------- |
  | **create** [options] <app-name>               | 프로젝트 생성                                                               |
  | **add** [options] <plugin> [pluginOptions]    | `npm install ` + 이미 생성된 프로젝트에 plugin 추가                         |
  | **invoke** [options] <plugin> [pluginOptions] | 이미 생성된 프로젝트에 plugin 추가                                          |
  | **inspect** [options] [paths...]              | 프로젝트의 webpack 설정 파일 자세히 보기                                    |
  | **serve** [options] [entry]                   | 개발 모드로 Application 실행 <br/>@vue/cli-service-global 전역 설치 필수    |
  | **build** [options] [entry]                   | 운영 모드로 Application 빌드 <br/>@vue/cli-service-global 전역 설치 필수    |
  | **ui** [options]                              | vue-cli ui 시작하기                                                         |
  | **init** [options] \<template> \<app-name>    | 다른 Template으로 프로젝트 생성 <br/> (legacy API, @vue/cli-init 설치 필수) |
  | **config** [options] [value]                  | 설정 자세히 보기 및 수정                                                    |
  | **upgrade** [semverLevel]                     | vue cli service / plugins 업그레이드 <br/>(default semverLevel: minor)      |
  | **info**                                      | 디버깅 정보 출력                                                            |

## CLI Service

- 복잡한 `webpack` 설정을 알아서 해줌
- `webpack-dev-server` 를 사용하여 Hot Module Replacement 가 가능하도록 함  
  (<g-emoji>👉</g-emoji> 코드를 저장하면 브라우저에 바로 반영)
- 사용 가능한 명령어는 `package.json` 에 정의되어 있음
  |command|desc|
  |-|-|
  |serve|개발서버 시작|
  |build|배포를 위한 빌드|
  |inspect|웹팩 내부 설정 보기|
  |lint|소스코드 작성 규칙 검사 및 수정|

> react의 `react-scripts` 와 유사한 개념  
> 참고자료 : [What exactly is the 'react-scripts start' command?](https://stackoverflow.com/questions/50722133/what-exactly-is-the-react-scripts-start-command)

## Vue CLI 설치

```bash
$> npm i -g @vue/cli
```

> Mac OS 사용 시 앞에 `sudo` 붙여야 함

```bash
$> vue -V
3.11.0
```

> <g-emoji>😎</g-emoji> `lecture_1` 에서 설치했다면 Pass 하셔도 됩니다.

# Vue.js 로 SPA 만들기

## Project 만들기

```bash
$> vue create shoppingmall
```

[![프로젝트 만들기](https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_4/VueCliCreateProject.png)](https://player.vimeo.com/video/367217922)

> `vue-cli 2.X` 에서는 `vue init <template-name> <project-name>` 명령어를 통해 프로젝트를 생성했었습니다.
>
> 관련된 내용은 [vue-cli v2 github](https://github.com/vuejs/vue-cli/tree/v2#vue-cli--) 를 참고하시기 바랍니다.

### preset 수동 설정하기

- 추가적인 설정을 해줘야 하기 때문에 `Manually select features` 선택
  <img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_4/make-project-1.png" />

- `Router`, `Vuex` 추가  
  (실제 개발 시 필수 선택 항목)
  <img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_4/make-project-2.png" />
- `history mode` 사용  
  (`vue-router`의 기본 모드는 `hash mode`임)
  <img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_4/make-project-3.png" />
- linter / formatter 는 `ESLint + Prettier`로 설정
  <img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_4/make-project-4.png" />

- 추가적인 lint 설정  
  (파일 저장 시 규칙에 맞게 코드가 작성되었는지 체크)
  <img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_4/make-project-5.png" />

- Babel, PostCSS, ESLint 등의 설정 파일은 전용 설정 파일 생성
  <img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_4/make-project-6.png" />

- 이렇게 프로젝트 설정한 값들을 저장할 수 있지만 여기서는 N 선택
  <img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_4/make-project-7.png" />

- 프로젝트 생성 완료
  <img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_4/make-project-8.png" />

## 실행하기

- `npm run serve` 실행  
  (만약에 8080 포트가 사용 중이라면 다른 포트 번호로 실행될 수 있음)
  <img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_4/make-project-9.png" />

- 브라우저에서 `http://localhost:8080` 접속
  <img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_4/make-project-10.png" />

## 스캐폴딩 된 소스코드 구조 분석

<img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_4/1.png" width=300px />

1. <g-emoji>📂</g-emoji>**node_modules** : 프로젝트를 빌드하는 데 필요한 모든 라이브러리가 저장되는 곳  
   (`npm install` 명령어를 실행하면 폴더가 생성됨)

2. <g-emoji>📂</g-emoji>**public** : Webpack을 통해 관리되지 않는 정적 Resource들을 모아두는 곳.

   > <g-emoji>⚠️</g-emoji> image나 font 같은 정적 Resource들은 **src/assets** 폴더에 넣어서 Webpack의 관리를 받게하는 것을 추천

3. <g-emoji>📂</g-emoji>**src/components** : 어플리케이션에서 사용되는 컴포넌트들을 모아두는 곳

4. <g-emoji>📂</g-emoji>**src/router** : Router 설정 파일들을 모아두는 곳 (Client-Side-Routing)

5. <g-emoji>📂</g-emoji>**src/store** : Vuex 설정 파일들을 모아두는 곳 (State 관리)

6. <g-emoji>📂</g-emoji>**src/views** : Application의 다른 뷰 또는 페이지에 대한 파일을 저장하는 위치

7. <g-emoji>📄</g-emoji>**App.vue** : 다른 컴포넌트를 포함하고 있는 최상위(root) 컴포넌트

8. <g-emoji>📄</g-emoji>**main.js** : 어플리케이션에 필요한 요소들을 Load하여 렌더링한 후 DOM에 마운트하게 하는 작업 수행

   ```js
   import Vue from "vue";
   import App from "./App.vue";
   import router from "./router";
   import store from "./store";

   Vue.config.productionTip = false;

   new Vue({
     router,
     store,
     render: (h) => h(App),
   }).$mount("#app");
   ```

9. <g-emoji>📄</g-emoji>**package.json** : 프로젝트에 필요한 package 들을 정의하고 관리해줌

> <g-emoji>✅</g-emoji> 프로젝트 생성시 Babel, PostCSS, ESLint 등의 설정 파일은 전용 설정 파일 생성하도록 선택하였기 때문에 `.eslintrc.js`, `babel.config.js`, `postcss.config.js` 파일이 생성됨

## Beautify 사용하기

> Vuetify = Vue + Design System(Material Design)

- Matterial Design 의 스타일 가이드에 맞춰 컴포넌트들을 미리 만들어 놓은 UI Library
- 다른 UI Library와의 비교했을 때 많은 장점을 보유하고 있음
  <img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_4/WhyVuetify.png">

### Browser support

- Vuetify는 Internet Explorer 의 오래된 버전을 지원 <g-emoji>❌</g-emoji>

  > Vuetify is a progressive framework that attempts to push web development to the next level. <span style="background-color:#ff00004f">In order to best accomplish this task, some sacrifices had to be made in terms of support for older versions of Internet Explorer.</span> This is not an exhaustive list of compatible browsers, but the main targeted ones.

  <img src="https://raw.githubusercontent.com/wooyoung85/vuejs-study/master/lecture/images/lecture_4/BrowserSupport.png">

- IE11 과 Safari9을 지원하기 위해서는 `babel-polyfill` 설치 후 약간의 설정 필요  
  (자세한 내용은 [공식문서](https://vuetifyjs.com/ko/getting-started/browser-support#ie-11-amp-safari-9-support)를 참고하시기 바랍니다~)

#### <g-emoji>⚠️</g-emoji> 유의사항

- **모든 브라우저를 지원해야 하는 시스템이라면 Vuetify를 사용하면 안됩니다!!**
- 사내 디자인 가이드가 있다면 굳이 Vuetify 같은 UI Library 를 사용할 필요는 없습니다 :)

> 저는 개인적인 프로젝트나 prototype을 개발할 때 매우 유용하게 사용하고 있습니다 <g-emoji>😎</g-emoji>

### Add Vuetify Plugin

```bash
$> vue add vuetify
```

### 주로 사용하는 기능 및 컴포넌트

- [Pre-mad layouts](https://vuetifyjs.com/ko/getting-started/pre-made-layouts)
- Style

  - [Colors](https://vuetifyjs.com/en/styles/colors)
  - [Display](https://vuetifyjs.com/en/styles/display)

- UI Components
  - [<g-emoji>⭐️</g-emoji>Grid system](https://vuetifyjs.com/ko/components/grids)
  - [Cards](https://vuetifyjs.com/ko/components/cards)
  - [Buttons](https://vuetifyjs.com/ko/components/buttons)
  - [Dialogs](https://vuetifyjs.com/en/components/dialogs)

> 화면 구성을 위한 기본적인 컴포넌트들은 거의 다 갖춰져 있으니 잘 찾아서 쓰시면 됩니다 :)

## 참고자료

[Vue CLI](https://cli.vuejs.org/)  
[Our Courses | Vue Mastery](https://www.vuemastery.com/courses/)
[[Vue] 개발환경 만들기 (without vue-cli)](https://velog.io/@kyusung/Vue-app-sfc-without-vue-cli)  
[Vue CLI 3.X 사용하기](https://velog.io/@skyepodium/Vue-CLI-3.X-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)  
[Vuetify 공식문서](https://vuetifyjs.com/ko/)
