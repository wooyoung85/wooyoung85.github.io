- Docker 설치

  - [📑 참고 자료](https://velog.io/@klasis/Windows-10%EC%97%90%EC%84%9C-Docker-for-Windows-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0)

- Font 다운로드

  - D2Coding : [🚀 Link](https://github.com/naver/d2codingfont/releases)
  - 배달의민족 한나체·주아체·도현체 : [🚀 Link](https://www.woowahan.com/#/fonts)

- Windows Terminal 설치

  - Microsoft Store 를 통해 설치

- Git 설정

  ```bash
  $> git config --global user.name "wooyoung"
  $> git config --global user.email wyseo@seegene.com
  $> git config --global credential.helper store
  ```

- `PowerShell` 자동완성 설정

  ```
  ps> Install-Module -Name PowerShellGet -Force
  ps> Install-Module PSReadLine -AllowPrerelease -Force

  # 에러가 난다면 아래 명령어 실행 후 다시 실행
  ps> Set-ExcutionPolicy Unrestricted
  ```

- PowerShell 버전 업데이트

  ```
  ps> $PSVersionTable.PSVersion
  ps> iex "& { $(irm https://aka.ms/install-powershell.ps1) } -UseMSI"
  ```

  > 관리자 권한으로 실행해야 함!!

- Powershell 폴더 강제 삭제

  ```
  ps> rm <path> -r -fo
  ```

- chocolatey(윈도우 패키지 매니저) 설치

  ```
  ps> Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
  ```

- Vim Install (필수 아님)
  ```
  ps> choco install vim
  ```
- Powershell 실행 시 한글깨짐 설정
  - `$PROFILE` (C:\Users\<Username>\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1) 에 아래 내용 추가
    ```
    $ENV:LC_ALL='C.UTF-8'
    [System.Console]::OutputEncoding = [System.Text.Encoding]::UTF8
    ```
