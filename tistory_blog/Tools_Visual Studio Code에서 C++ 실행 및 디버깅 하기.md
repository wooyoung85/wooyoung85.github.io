## 빠르게 실행하기
- code runner extension 설치 후 바로 실행 가능
- 마우스 우클릭 >> Run Code (단축키 : `control` + `option` + `N`)

## 디버깅하기

#### 1. 빌드 설정
- 터미널 >> 빌드 작업 실행 (단축키 : `cmd` + `shift` + `B`)
- others 선택
- tasks.json 생성
- label, command 변경 후 빌드 실행  
    > GNU 컴파일러(gcc)로 main.c 를 컴파일해서 main.out 을 만드는 작업을 build 라고 이름 붙이기

    ```json
    {
        "version": "2.0.0",
        "tasks": [
            {
                "label": "build",
                "type": "shell",
                "command": "gcc -g main.c -o main",
            },
        ]
    }
    ```
- group, problemMatcher 추가
    > 해당 작업이 build 그룹에 속하게 하고, 컴파일 하면서 나는 에러를 볼 수 있게 problemMatcher 속성을 설정해 줌 
    ```json
    {
        "version": "2.0.0",
        "tasks": [
            {
                "label": "build",
                "type": "shell",
                "command": "gcc -g main.c -o main",
                "group": {
                    "kind": "build",
                    "isDefault": true
                },
                "problemMatcher":"$gcc"
            },
        ]
    }
    ```

#### 2. 디버깅 설정
- 디버깅 탭에서 Add Configuration
- c++(gdb) 선택
- program 변경, preLaunchTask 추가, externalConsole 변경
    ```json
    {
        "version": "0.2.0",
        "configurations": [
            {
                "name": "(lldb) Launch",
                "type": "cppdbg",
                "request": "launch",
                "preLaunchTask": "build",
                "program": "${workspaceFolder}/main",
                "args": [],
                "stopAtEntry": false,
                "cwd": "${workspaceFolder}",
                "environment": [],
                "externalConsole": false,
                "MIMode": "lldb"
            }
        ]
    }
    ```

## 참고자료
[VS Code: C++ Development With Visual Studio Code](https://www.youtube.com/watch?v=X7CXjKGi_ro)