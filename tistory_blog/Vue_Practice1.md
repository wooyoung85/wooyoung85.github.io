
## Component 를 생성
.vue에서 js 로직을 기술하기 위해 `<script>`를 만드는 방법은 기본적으로 3 가지 정도가 있음

# Getting Started

## Installation
### CDN
- [unpkg](https://unpkg.com/vue/dist/vue.js) 확인
```html
<script src="https://unpkg.com/vue/dist/vue.js></script>
```

### NPM
```bash
$ npm install vue
```
### Vue-CLI 설치
```bash
$ npm i -g @vue/cli
```

## First Project Build
>[jsfiddle](https://jsfiddle.net/) : Web Editor로 간단하게 JavaScript, HTML, CSS를 포함한 코드를 실행해 볼 수 있다.

### 실습 1

```html
<script src="https://unpkg.com/vue/dist/vue.js"></script>

<div id='app'>
  <p>{{ title }}</p>
</div>
```

```javascript
var app = new Vue({
  el: '#app',
  data: {
    title: 'Hello Vue!'
  }
})
```

### Vue Instance

### Expression
```html
{{ product + '?' }}
{{ firstName + ' ' + lastName }}
{{ message.split('').reverse().join('') }}
```

## Attribute Binding

### v-bind
Dynamically binds an attribute to an expression.


## Conditional Rendering

### v-if, v-else-if, v-else, v-show

## List Rendering

### v-for

### Event Handling

### Style Binding



