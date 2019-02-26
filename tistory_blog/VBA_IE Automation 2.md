> [VBA로 Internet Explorer 작업 자동화 하기](https://wooyoung85.tistory.com/42) 에서 VBA 개발을 위한 준비를 끝냈다고 가정하고 Post를 작성하였습니다.

# TL;DR
- 모듈에 아래 코드 입력 후 `실행(F5)`
- 테스트 웹페이지가 열린 후 값이 입력되고 제출 버튼을 클릭하게 됨
    ```vb
    Sub IE_Automation()

        ' 변수 선언
        Dim i As Long
        Dim o As Object
        Dim IE As Object
        Dim objCollection As Object
        Dim objInputElement As Object
        Dim objSelectElement As Object
        Dim objButtonElement As Object
        
        ' Internet Explorer 열기
        Set IE = CreateObject("InternetExplorer.Application")
        IE.Top = 0
        IE.Left = 0
        IE.Height = 1000
        IE.Width = 1600
        IE.Visible = True
        IE.navigate "C:\Users\test\Desktop\main.html"
        
        ' 페이지 로드 대기
        Do While IE.Busy
            Application.Wait DateAdd("s", 1, Now)
        Loop
        Debug.Print "페이지 로드 완료"
        
        ' textbox에 값 입력
        Set objInputElement = IE.document.getElementsByName("name")(0)
        objInputElement.Value = "문타리"
        
        ' selectbox 선택하기
        Set objSelectElement = IE.document.getElementById("age")
        For Each o In objSelectElement.Options
            If o.Value = "30" Then
                o.Selected = True
                Exit For
            End If
        Next
        
        ' button 클릭
        Set objCollection = IE.document.getElementsByTagName("button")
        i = 0
        While i < objCollection.Length
            If objCollection(i).innerText = "제출" Then
                Set objButtonElement = objCollection(i)
            End If
            i = i + 1
        Wend
        objButtonElement.Click
        
        ' Internet Explorer 닫기
        IE.Quit
        
    End Sub
    ```

# Modal 창과 Alert 창 처리
Internet Explorer 자동화 코드를 작성하다 보면 `Alert 창(웹 페이지 메시지)`과 `Modal 창(웹 페이지 대화 상자)` 때문에 더 이상 진행이 어려울 때가 있다.  
이를 해결하기 위한 방법을 정리해 보도록 하겠다.

## javascript 함수 재정의  
- 버튼을 클릭하기 전에 `window.alert`와 `window.showModalDialog`를 재정의
- modal 창을 사용할 경우 보통 `val retVal = window.showModalDialog('...')` 이런 형태로 많이 사용하기 때문에 `IE 개발자도구(F12)`의 디버그 탭을 최대한 활용하여 웹 페이지의 소스코드를 분석한 후 알맞게 javascript 함수 재정의 하면 된다.
    ```vb
    With IE.document.parentWindow
        .execScript "window.alert = function(){return true;};", "JScript"
        .execScript "window.showModalDialog = function(){return 'Success';};", "JScript"
    End With
    ```
## VBS 활용
위 방법으로 해결이 안될 때는 VBS(Visual Basic Script)를 사용하여 해결 할 수 있다.
- `modal_close.vbs`
    ```vb
    Set wShell = CreateObject("WScript.Shell")

    Do
        ret = wShell.AppActivate("웹 페이지 대화 상자")
    Loop Until ret = True

    WScript.Sleep 500

    If ret = True Then
        WScript.Sleep 10
        wShell.SendKeys "%{F4}"
    End If
    ```
- `modal_close.vbs` 실행하기
    ```vb
    Shell "wscript ""C:\Users\test\Desktop\modal_close.vbs"""
    ```

> **위 예제의 경우에는 버튼을 클릭하기 전에 javascript 함수 재정의하거나 vbs 파일을 실행해야 함**
> ```vb
> ' selectbox 선택하기
> ...
>
> ' javascript 함수 재정의
> 'With IE.document.parentWindow
> '    .execScript "window.alert = function(){return true;};", "JScript"
> '    .execScript "window.showModalDialog = function(){return 'Success';};", "JScript"
> 'End With
>
> ' vbs 실행하기
> Shell "wscript ""C:\Users\awood\Desktop\modal_close.vbs"""
> Shell "wscript ""C:\Users\awood\Desktop\alert_close.vbs"""
> 
> ' button 클릭
> ...
> ```

## 참고자료
[Automate Internet Explorer (IE) Using VBA](https://www.automateexcel.com/vba/automate-internet-explorer-ie-using/)  
[VBA Excel Macro Stops Working When a Dialog Box Pop Up Appears - InternetExplorer.Application](https://youtu.be/G-2khNFYQl8)