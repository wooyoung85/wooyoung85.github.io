> 윤인성 님의 [머신러닝, 딥러닝 실전 개발 입문](http://wikibook.co.kr/python-machine-learning/) 책을 보면서 실습 중에 Trouble Shooting 했던 내용을 정리하였습니다.


## Cron 실습 중 Ubuntu 관련 문제

우분투 docker 이미지가 Minimize 된 형태로 제공되기 때문에 ubuntu:lateast 혹은 ubuntu:16.04 이미지를  pull 해서 컨테이너를 띄우면 sudo, cron 등 기본적으로 필요한 패키지들이 설치가 안되어 있음


### 실습에 필요한 패키지 설치

```
$ apt-get update
$ apt-get install sudo cron nano
$ apt-get install -y rsyslog
``` 

### cron 서비스 확인 및 시작

```
$ ps aux | grep cron          -- pid 확인
$ pgrep cron                  -- pid 확인
$ service cron status         -- 서비스 상태 확인
$ sudo service cron start     -- 서비스 시작
```

### hexdump
docker ubuntu에 안 깔려서 python의 hexdump 패키지 설치 후 확인

```
$ pip3 install hexdump
```


## 환경 설정

### 한글 인코딩 설정

1. docker run 할 때 설정하기
    ```
    $ docker run -i -t -v <마운트 할 폴더 경로>:<마운트 된 폴더명> --name <컨테이너 이름> -e ko_KR.UTF-8 -e PYTHONIOENCODING=utf-8 <docker 이미지 이름> /bin/bash
    ```
    - -e ko_KR.UTF-8 \                --> language export
    - -e PYTHONIOENCODING=utf-8 \     --> python encoding export

2. 실행 중인 container에 attach해서 직접 설정하기
    ```
    $ export LANG="ko_KR.UTF-8"
    $ export PYTHONIOENCODING=utf-8
    ```

### datetime 설정

```
$ apt-get install tzdata
$ tzselect --> 4번-Asia 23번-Korea(South) 1번-Yes
```

## ubuntu 환경에서 pip3 install 명령어 실행 시 오류날 경우 

- 분명 처음엔 문제 없었는데 아무래도 실습하다 Locale 관련 설정을 건드린 듯..ㅠㅠ

- 현상

    ```
    $ pip3 install pyyaml
    Traceback (most recent call last):
    File "/usr/bin/pip3", line 11, in <module>
        sys.exit(main())
    File "/usr/lib/python3/dist-packages/pip/__init__.py", line 215, in main
        locale.setlocale(locale.LC_ALL, '')
    File "/usr/lib/python3.5/locale.py", line 594, in setlocale
        return _setlocale(category, locale)
    locale.Error: unsupported locale setting
    ```

- 해결방법  
    ```
    $ export LC_ALL=C
    ```

## 윈도우에서 pyyaml 설치하는 방법

- PowerShell에서 pip install 명령어를 날리면 무지막지하게 오류가 남

    ```
    PS C:\Users\awood> pip install PyYAML
    ```

- 아래 URL로 들어가서 파일 리스트 중 zip 파일 다운로드  
[https://pypi.python.org/pypi/PyYAML](https://pypi.python.org/pypi/PyYAML)
- 압축 해제 후 해당 폴더에서 PowerShell을 열어서 setup.py 파일 실행  
`PyYAML-3.12> python setup.py install`

## 엑셀 파일 읽기
- 엑셀 파일에서 콤마를 빈문자열('')로 치환 후 숫자 & 쉼표 셀 서식으로 변경 (~~이것 때문에 삽질을 겁나게 함~~)

- 필요없는 줄을 날리려면 아래와 같이 지워줘야 함  

    ```python
    import openpyxl

    # 엑셀 파일 열기く --- (※1)
    filename = "stats_104102.xlsx"
    book = openpyxl.load_workbook(filename)

    # 맨 앞의 시트 추출하기 --- (※2)
    sheet = book.worksheets[0]

    # 시트의 각 행을 순서대로 추출하기 --- (※3)
    data = []
    for row in sheet.rows:
        data.append([
            row[0].value,
            row[9].value
        ])

    # *****필요없는 줄(헤더, 연도, 계) 제거하기*****
    del data[0]
    del data[0]
    del data[0]

    # 데이터를 인구 순서로 정렬합니다.
    data = sorted(data, key=lambda x:x[1])

    # 하위 5위를 출력합니다.
    for i, a in enumerate(data):
        if (i >= 5): break
        print(i+1, a[0], int(a[1]))
    ```

## sorting 예제
- 제곱수가 가장 작은 순서대로 Sorting

    ```python
    a = [-1, -8, 3, -4, 2, 5, -7]
    a.sorted(key=lambda x : x*x, reverse=False) 
    print a
    ```

## Pandas를 이용해 엑셀 파일 읽기

- Warning 메세지 발생 시 아래와 같이 코드 수정  
("The `sheetname` keyword is deprecated, use `sheet_name` instead")

    ```python
    import pandas as pd
    # 엑셀 파일 열기 --- (※1)
    filename = "stats_104102.xlsx" # 파일 이름
    sheet_name = "stats_104102" # 시트 이름
    book = pd.read_excel(filename, sheet_name=sheet_name, header=1) # 첫 번째 줄부터 헤더
    # 2017년 인구로 정렬 --- (※2)
    book = book.sort_values(by=2015, ascending=False)
    print(book)
    ```

## MySQL 서비스 시작 안 될 때
 
```
chown -R mysql: /var/lib/mysql
```

**chown [옵션] [소유자:소유그룹] [파일]**  
파일의 소유자나 소유그룹을 변경하기 위한 명령어이다.

**[옵션]**  
- -c: 변경된 파일만 자세하게 보여준다.
- -f: 변경되지 않은 파일에 대해서 오류 메시지를 보여주지 않는다.
- -v: 작업상태를 자세히 보여준다.
- -R: 경로와 그 하위 파일들을 모두 변경한다.

**※ 소유그룹을 입력하지 않으면 소유자만 변경됨.**


## 참고자료

[머신러닝, 딥러닝 실전 개발 입문](https://www.youtube.com/watch?v=vGrd5bSoBs8&list=PLBXuLgInP-5m_vn9ycXHRl7hlsd1huqmS&index=1)