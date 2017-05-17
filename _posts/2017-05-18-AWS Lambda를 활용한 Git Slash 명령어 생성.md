# AWS Lambda를 활용한 Git Slash 명령어 생성

AWS Lambda와 API GateWay를 활용한 Slack의 Slash 명령어를 만들어 보려고 합니다.
(오로지 실습하는데만 집중하겠습니다 ^^)

순서는 아래와 같이 진행하려고 합니다.
>1. Slack의 App과 Slash 명령어 생성
2. AWS Lambda 생성
3. API Gateway 생성
4. 연동


## Slack의 App과 Slash 명령어 생성
###### Slack에 가입 후 Cloud Study라는 채널에 입장한 상태로 시작합니다~

![post_1_1](/images/post_1_1.png)
- 왼쪽 상단에 이름을 누르면 나오는 Apps & integrations 메뉴 클릭!

![post_1_2](/images/post_1_2.png)
- 오른쪽 상단 Build 버튼 클릭!

![post_1_3](/images/post_1_3.png)
- Start Building 버튼 클릭!

![post_1_4](/images/post_1_4.png)
- AppName을 입력하시고
- 여러 팀(채널)에 속해 있다면 만든 App을 적용시킬 팀(채널)을 선택하시면 됩니다.
- Create App 버튼 클릭!

![post_1_5](/images/post_1_5.png)
- Slash Commands 선택!
- 다음 페이지에서 Create New Command 버튼 클릭!

![post_1_6](/images/post_1_6.png)
- command 명 입력   ex) /wiki
- Request URL은 API가 뱉어내는 URL을 나중에 입력
- Short Description은 짧게 설명을 샤샤샥
- Usage Hint   ex) [Search Text]


## AWS Lambda 생성


## API Gateway 생성
