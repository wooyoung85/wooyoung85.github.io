> `Internet Explorer`를 꼭 써야만 할 경우(ActiveX 기반 시스템, 사내 시스템 등등)  
VBA를 활용하여 작업들을 자동화 할 수 있는 방법에 대해 정리해 보았습니다.

# VBA 개발을 위한 필수 조건
- 윈도우 OS 설치된 PC
- Excel 이 꼭 있어야 함
- .NET Framework(Window PC라면 당연히 가 설치되어 있음)

> 저는 Windows 10 Pro와 Microsoft Office Professinal Plus 2016 사용 중 입니다.

# 개발 환경 Setting
- Excel 개발 도구 활성화  
`파일` >> `옵션` >> `리본 사용자 지정` >> `개발 도구` 체크 >> 확인
- 모듈 추가하기  
`개발 도구` >> `Visual Basic` >> Sheet1에 우클릭 >> `삽입` - `모듈` 클릭
- Microsoft HTML Object Library 참조 추가  
`도구` >> `참조` >> `Microsoft HTML Object Library` 체크 >> 확인

# 테스트 페이지
- Form 에 입력한 값을 Modal 창에서 확인하는 간단한 테스트 페이지  
- IE에 테스트 페이지가 안전하다고 알리기 위해 아래 코드 추가 하였음
    ```html
    <!-- saved from url=(0014)about:internet -->
    ```
- main.html
    ```html
    <html>

    <head>
        <meta charset="UTF-8">
        <!-- saved from url=(0014)about:internet -->
        <script type="text/javascript">
            function fntest() {
                var obj_name = document.getElementsByName("name")[0];
                var obj_age = document.getElementById("age");
                var args = new Array();

                args['name'] = obj_name.value;            
                args['age'] = obj_age.options[obj_age.selectedIndex].text;

                var msgDialog = window.showModalDialog('modal.html', args, 
                'dialogWidth:320px; dialogHeight:200px; center:yes; help:no; status:no; scroll:no; resizable:no');
                alert(msgDialog);
                return msgDialog;
            }
        </script>
        <style>
            table {
                border: 2px solid black
            }

            tr {
                outline: 1px solid black
            }

            .bg_text {
                background-color: antiquewhite
            }
        </style>
    </head>

    <body>
        <h1>테스트 페이지</h1>
        <table>
            <tbody>
                <tr>
                    <td>이름</td>
                    <td><input type="text" name="name" class="bg_text" /></td>
                </tr>
                <tr>
                    <td>나이</td>
                    <td>
                        <select id="age">
                            <option value="">선택</option>
                            <option value=10>10</option>
                            <option value=20>20</option>
                            <option value=30>30</option>
                            <option value=40>40</option>
                            <option value=50>50</option>
                            <option value=60>60</option>
                        </select>
                    </td>
                </tr>
                <tr>
                    <td>취미</td>
                    <td><input type="text" name="name" class="bg_text"></td>
                </tr>
            </tbody>
        </table>
        <button type="submit" onclick="fntest()">제출</button>
    </body>

    </html>
    ```

- modal.html
    ```html
    <html>

    <head>
        <meta charset="UTF-8">
        <!-- saved from url=(0014)about:internet -->
        <base target="'_self'">
    </head>

    <body style="margin:0px">
        <h1>테스트 모달창</h1>
        <label id='lbl_name'></label><br />
        <label id='lbl_age'></label><br />
        <button onclick="window.close()">닫기</button>
    </body>
    <script>
        document.getElementById('lbl_name').innerHTML = window.dialogArguments['name'];
        document.getElementById('lbl_age').innerHTML = window.dialogArguments['age'];

        var ret_val = 'GOOD';
        window.returnValue = ret_val;
    </script>

    </html>
    ```

# Internet Explorer 열기
```vb
Sub IE_Automation()
    Dim IE As Object
    Set IE = CreateObject("InternetExplorer.Application")
    IE.Top = 0
    IE.Left = 0
    IE.Height = 1000
    IE.Width = 1600
    IE.Visible = True

    ' 테스트 페이지 경로 입력
    IE.navigate "C:\Users\test\Desktop\main.html"
End Sub
```

> 매크로가 포함된 파일 저장 시 꼭 `엑셀 매크로 사용 통합 문서(.xlsm)` 형식으로 저장해야 합니다 :)