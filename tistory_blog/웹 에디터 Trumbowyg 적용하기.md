## Trumbowyg 적용 방법
- pom.xml에 trumbowyg dependency 추가
```xml
<dependency>
<groupId>org.webjars.bower</groupId>
<artifactId>trumbowyg</artifactId>
<version>2.4.1</version>
</dependency>
bootstrap과 jquery 를 먼저 설정한 후 trumbowyg 경로 설정
<link href="/webjars/bootstrap/3.3.7-1/css/bootstrap.min.css" rel="stylesheet">
<link href="/webjars/trumbowyg/2.4.1/dist/ui/trumbowyg.min.css" rel="stylesheet">
........

<script src="/webjars/jquery/3.2.1/jquery.min.js"></script>
<script src="/js/bootstrap.min.js"></script>
<script src="/webjars/trumbowyg/2.4.1/dist/trumbowyg.min.js"></script>
<script src="/webjars/trumbowyg/2.4.1/dist/langs/ko.min.js"></script>
```
- 에디터 적용을 위한 class명 설정과 한글 패치 적용
```javascript
$('.edit').trumbowyg({
lang : 'kr'
});
```
- textarea 태그 class에 edit 추가
```html
<textarea class="form-control edit" id="contents" name="contents" rows="5"></textarea>
```
- plugin을 추가하면 파일 업로드도 가능


## 참고사이트
[A lightweight WYSIWYG editor](https://alex-d.github.io/Trumbowyg/)

[![자바스크립트 텍스트 에디터 적용하기](http://img.youtube.com/vi/oOFNh0KM7kU/0.jpg)](https://www.youtube.com/watch?v=oOFNh0KM7kU?t=0s) 자바스크립트 텍스트 에디터 적용하기