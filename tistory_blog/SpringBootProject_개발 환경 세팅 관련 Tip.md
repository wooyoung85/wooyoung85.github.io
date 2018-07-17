## 1. 메이븐 프로젝트 Install 시 오류
- 프로젝트에서 마우스 우클릭 > maven > update project
- 메이븐 리소스 저장소 .m2 폴더를 삭제 후 다시 다운로드<br/>
  (자세한 경로는 `Windows > preferences > maven > user settings` 에서 확인)

## 2. Could not find or load main class org.apache.maven.wrapper.MavenWrapperMain
- .gitignore 파일에 셋팅된 항목 중 *.jar 가 있어서 github에 저장이 안 됨;;
- 해당경로에 maven-wrapper.jar 파일 넣어주면 해결됨

> Hello! I ran into this problem also, it worked fine at first, but if cloned the project on a new folder I got the <code>Error: Could not find or load main class org.apache.maven.wrapper.MavenWrapperMain,</code> for me the problem was that the Github Desktop application did not recognized the <code>!.mvn/wrapper/maven-wrapper.jar</code>line of the <code>.gitignore</code> file, so the wrapper was not being versiones. I solved it doing a commit of the <code>.mvn/wrapper/maven-wrapper.jar</code> file using the command line.

[참고한 사이트](https://github.com/jhipster/jhipster-core/issues/70)

## 3. 메이븐 컴파일 오류 : Installed_JREs 수정하기
> [ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.0:compile (default-compile) on project pcsell: Compilation failure<br/>
[ERROR] No compiler is provided in this environment. Perhaps you are running on a JRE rather than a JDK?<br/>
[ERROR] -> [Help 1]<br/>
[ERROR]<br/>
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.<br/>
[ERROR] Re-run Maven using the -X switch to enable full debug logging.<br/>
[ERROR]<br/>
[ERROR] For more information about the errors and possible solutions, please read the following articles:<br/>
[ERROR] [Help 1] 
>

- Preference > JAVA > Installed JREs 에서 JDK 선택
- JDK가 없다면 Add 버튼 눌러서 추가 후 선택

- Preference > JAVA > Installed JREs > Execution Environments 에서 JDK 선택

## 4.한글깨짐 현상 관련 (UTF-8)
- 메뉴 > Windows > preferences 로 이동
- Web 항목 중 아래 3개를 모두 UTF-8로 설정한다.

- General -> Workspace의 Text file encoiding 을 UTF-8로 바꿔준다.

- HTML5 페이지에 아래 코드 삽입
~~~html
<meta charset="utf-8">
~~~
- XHTML 페이지는 아래 코드 삽입
~~~html
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
~~~

## 5. STS 및 GRADLE SUPPORT PLUGIN 설치
- Dashboard 화면에서 “IDE EXTENSIONS”를 click한 후 Gradle Support 설치