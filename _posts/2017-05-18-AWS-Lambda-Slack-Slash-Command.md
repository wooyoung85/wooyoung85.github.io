---
layout: post
title: "AWS Lambda를 활용한 Slack Slash 명령어 생성"
description: "AWS 서버리스 프레임워크 실습"
date: 2017-05-17
tags: [AWS Lambda, API GateWay, Slack]
comments: true
share: true

sitemap :
  changefreq : weekly
  priority : 1.0
---
AWS Lambda와 API GateWay를 활용한 Slack의 Slash 명령어를 만들어 보려고 합니다.

**진행 순서**
>1. Slack의 App과 Slash 명령어 생성
2. AWS Lambda + API Gateway 생성
3. 연동


## Slack의 App과 Slash 명령어 생성
###### Slack에 가입 후 Cloud Study라는 채널에 입장한 상태로 시작합니다~
슬랙은 메신저 뿐만 아니라 다양한 통합 기능을 제공하고 있습니다. (Web Hooking, Slash Command등)

![post_1_1](/images/post_1_1.png)
- 왼쪽 상단에 이름을 누르면 나오는 Apps & integrations 메뉴 클릭!

![post_1_2](/images/post_1_2.png)
- 오른쪽 상단 Build 버튼 클릭!

![post_1_3](/images/post_1_3.png)
- Start Building 버튼 클릭!

![post_1_4](/images/post_1_4.png)
- AppName을 입력하시고
- App을 적용시킬 팀을 선택하시면 됩니다.
- Create App 버튼 클릭!

![post_1_5](/images/post_1_5.png)
- Slash Commands 선택!
- 다음 페이지에서 Create New Command 버튼 클릭!

![post_1_6](/images/post_1_6.png)
- command 명 입력   <sup>ex) /wiki</sup>
- **<u>Request URL은 AWS API Gateway가 뱉어내는 URL을 나중에 입력</u>**
- Short Description은 짧게 설명을 샤샤샥 작성
- Usage Hint   <sup>ex) [Search Text]</sup>


## AWS Lambda 생성
###### 아마존 클라우드 프리티어 사용 중입니다~

아마존 클라우드 로그인 후 대쉬보드 화면으로 GO GO

![post_1_7](/images/post_1_7.png)
- AWS에서 제공하는 서비스는 엄청 많으니까 검색창에 lambda라고 검색
- 다음 페이지에서 Create New Function 버튼 클릭!

![post_1_8](/images/post_1_8.png)
- Select blueprint는 샘플로 lambda를 구성해 놓은 것이고,
- Configure triggers는 다른 AWS 서비스에서 코드를 자동으로 트리거하도록 설정하는 것 입니다.<br>
<sup>적용사례) DynamoDB 테이블의 데이터 수정에 응답하는 애플리케이션을 구축할 수 있습니다.</sup>
- Configure function을 클릭하시면 아래와 같은 화면이 보이게 됩니다.

![post_1_9](/images/post_1_9.png)
- Name 입력하시고
- Description에 간단한 설명과
- Runtime 환경 설정해 주시면 됩니다
- Code entry type은 코드를 어떻게 등록할 것인지 선택해야 합니다.
 1. [배포 패키지 만들기](http://docs.aws.amazon.com/ko_kr/lambda/latest/dg/deployment-package-v2.html)를 통해 로컬PC에서 작업한 소스를 업로드 하시거나
 2. Amazon S3 bucket에 올려놓은 코드가 있다면 그걸 쓰셔도 되고 
 3. Edit code inline로 직접 작성하셔도 됩니다.
- 여기서는 Node JS 런타임 환경과 Edit code inline을 선택하겠습니다^^
아래 코드를 입력해 주세요~ 
슬랙 슬래시 명령어에서 날려준 text를 받아다가 wiki 검색 URL을 리턴하는 코드입니다.

```nodejs
// The entry point of this lambda function.
exports.handler = function(event, context) {
  console.log(event);

  context.succeed({
    "response_type": "in_channel",
    "text": "I found some about \"" + event.text + "\"",
    "attachments": [
        {
            "text": "For more info please visit to \nhttps://ko.wikipedia.org/wiki/" + event.text,
            "color": "#7CD197"
        }
    ]
  });

};
```
- Existing role은 service-role/myRoleName 선택 후 Next 버튼 클릭!
- 다음 화면에서 Create fuction 버튼 클릭!

![post_1_10](/images/post_1_10.png)
- function을 만든 후 Test 버튼 클릭하면 아마 위 코드에서 선언한 event.text 값이 undefined 로 결과 로그가 나오게 될 겁니다.
당황하지 마시고 Actions 드롭다운박스를 클릭해서 Congigure test event를 클릭해 주세요.

![post_1_11](/images/post_1_11.png)
- 테스트 할 이벤트를 위와 같이 수정 후 Save ant test 하면 결과 로그가 제대로 나올겁니다.

## API Gateway 생성
AWS Lambda는 혼자서는 어떤 기능을 하기 힘들기 때문에 이제 API Gateway와 연결해서 Slack에서 가져다 쓸 수 있도록 작업해 보겠습니다.
![post_1_12](/images/post_1_12.png)
- 대쉬보드로 가서 API Gateway 검색
- 다음페이지에서 Create API 버튼 클릭!
- 그 다음페이지에서 API Name과 Description을 쓰시고 Create API 버튼 클릭!

![post_1_13](/images/post_1_13.png)
![post_1_14](/images/post_1_14.png)
- Actions 드롭다운박스를 클릭해서 Create Resource
- 생성된 Resource를 클릭한 상태로 Actions 드롭다운박스를 클릭해서 Create Method 중 post Method를 생성
(오른쪽 체크버튼을 누르면 생성완료 됨)

![post_1_14](/images/post_1_15.png)
- Lambda Region은 AWS Lambda >> Functions 에서 해당 Function의 ARN 정보를 보면 나와있습니다
<sup>그냥 하나씩 Region을 선택해보는게 더 빠를수도 있습니다.</sup>
- Setup을 마치면 Resource 트리에서 post Method를 클릭하면 Method Execution 화면이 보이게 됩니다. 

![post_1_16](/images/post_1_16.png)
- API Gateway를 통해 Lambda로 들어가는 과정 중 “Integration Request”에서 일부 수정이 필요합니다.
([폼 방식의 POST 데이터를 JSON으로 변환시키는데 필요한 맵핑 루틴](https://gist.github.com/sjoonk/20ae13e5cd8be88e9824e3bad11b2859) 을 Body Mapping Templates 에 추가)
위 이미지 보고 잘 따라해 주세요~

<code>Slack에서 명령어를 호출하면 "application/x-www-form-urlencoded" Content-Type을 사용하여 폼 post 방식 전송을 하기 때문에
이를 JSON 형태로 변환하는 작업이 필요합니다.</code>

**마지막으로 Actions 드롭다운박스를 클릭해서 Deploy API 해 주셔야 수정한 것들이 적용됩니다.**

## 연결
Lambda와 API Gateway가 잘 생성됐으면 맨 왼쪽 Tree에서 API >> Stages 에 가면
Invoke URL 정보를 보실 수 있습니다.

이 URL을 복사해서 아까 맨 처음에 Slack 명렁어 생성시 비워뒀던 Request URL에 채워넣고,
Slack 채팅창으로 가서 생성한 명령어가 사용가능한지 확인 해 보시면 됩니다~
<br><br>
블로그도 처음인데 마크다운 문법을 사용해서 하다보니 너무 힘들었네요..ㅠㅠ

---
**참고한 블로그** <br>
[AWS Lambda와 API Gateway로 Slack Bot 만들기](http://www.usefulparadigm.com/2016/04/06/creating-a-slack-bot-with-aws-lambda-and-api-gateway/)<br>
[AWS API GatewayでContent-Type:application/x-www-form-urlencoded のPOSTデータを受け取り JSONに変換する](http://qiita.com/durosasaki/items/83af014aa85a0448770e)<br>
[AWS Lambda를 이용해서 GitHub과 Slack 연동하기](http://blog.weirdx.io/post/27097)<br>
[[AWS]서버없이 Lambda와 API Gateway로 서버API 만들기](http://gun0912.tistory.com/59)
