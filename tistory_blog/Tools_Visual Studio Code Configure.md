# 블로깅 관련 설정

### 확장팩 설치
- `Copy Markdown as HTML` 설치  
마크다운으로 작성 후 HTML로 변환하여 티스토리 블로그에 게시하기 위해 설치함

### User Settings
- 마크다운 문서 HTML로 변환 시 HTML TAG가 잘 변환될 수 있도록 설정
    ```json
    {
        "copyMarkdownAsHtml.html": true
    }
    ```
### 실행 방법
- `F1` 키 누른 후 `Markdown: Copy as HTML` 명령을 실행하면 클립보드에 변환된 HTML이 복사됨


# 파이썬 관련 설정
- 머신러닝, 딥러닝 관련 개발이 주 목적이라면 아나콘다를 설치해서 환경 구성하는 것이 좋다.

- [아나콘다 설치](https://www.anaconda.com/download/) 할 때 Add Anaconda to my PATH environment variable 체크를 안 했다면 `C:\Users\{사용자계정}\AppData\Local\Continuum\anaconda3` 을 환경변수에 추가

- 기존에 설치한 파이썬이 환경변수에 추가되어 있다면 삭제해야 함  
삭제 후 아래와 같이 아나콘다의 파이썬을 사용하는지 확인
    ```cmd
    $ python
    Python 3.6.5 |Anaconda, Inc.| (default, Mar 29 2018, 13:32:41) [MSC v.1900 64 bit (AMD64)] on win32
    Type "help", "copyright", "credits" or "license" for more information.
    >>>
    ```

### 확장팩 설치
- `Ananconda Extension Pack` 설치  
아나콘다 환경을 Visual Studio Code 상에서 사용하기 위한 확장팩

### 기본 빌드 작업 구성
- 메뉴 > 작업 > 기본 빌드 작업 구성
    ```json
    {
        "version": "2.0.0",
        "tasks": [
            {
                "label": "python",
                "command": "python",
                "args": [
                    "${file}"
                ],
                "group": {
                    "kind": "build",
                    "isDefault": true
                }
            }
        ]
    }
    ```

### 실행 방법
- python 코드 작성 후 `Ctrl + Shift + B` 로 빌드하면 python 코드 실행되는지 확인
    ```python
    print('Build Task Setting!')
    ```