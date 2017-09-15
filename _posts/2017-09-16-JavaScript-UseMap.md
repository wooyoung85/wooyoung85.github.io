---
layout: post
title: "javascript usemap 사용법"
description: "javascript usemap 사용법"
date: 2017-09-16
tags: [javascript, usemap]
comments: true
share: true

sitemap :
  changefreq : weekly
  priority : 1.0
---
이미지 위에 버튼을 만들어 주기 위해 usemap 속성을 사용해 만들어 줬다.
간단하지만 나중에 다시 보기 위해서 저장~ ㅋ

- 이미지가 너무 크면 안됨
- 정확하게 위치를 잡기위해 화면에 이미지 크기가 100% 다 나올 수 있도록 해야함
  (이미지 크기를 줄이거나 이미지 표시되는 부분의 영역을 이미지 크기에 맞춰줘야함)
- area는 여러개 추가 가능

```javascript
<img src="/images/myimage.png" usemap="#imgmap" />
	<map name="jobMap" id="jobMap">
    	<area shape="rect" coords="820,86,909,111" alt="버튼" title="버튼" onclick="window.open('http://www.naver.com');" href="#" />
    </map>
```
