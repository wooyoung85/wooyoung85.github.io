## 아나콘다란? <span style="font-size:11pt">[Distribution-Anaconda](https://www.anaconda.com/distribution/)</span>

- 엄청나게 많은(1400+) 파이썬과 R 데이터 사이언스 패키지들을 설치하고, 패키지, 의존성, 환경에 대한 부분들을 관리한다.
- 무료이고 오픈소스

## 설치파일 다운로드
[https://www.anaconda.com/download/](https://www.anaconda.com/download/)

## 환경변수 설정

Add Anaconda to my PATH environment variable 체크하기
 

## 설치 확인 및 업데이트

```
conda --version

conda update conda
```

## 아나콘다 명령어

### 새로운 가상환경 생성

- "python3"라는 이름의 파이썬 버전3를 사용하는 가상환경을 생성하기
- 뒤에 `anaconda` 를 붙여서 아나콘다 배포를 구성하는 모든 파이썬 패키지를 포함하도록 conda 환경 구성

```
conda create -n "python3" python=3 anaconda
```

### 가상환경 목록 보기

```
conda env list
```

### 가상환경 활성화 / 비활성화

```
activate <환경명>
deactivate
```

**※ 위 명령어가 실행이 안 될 경우**

```
source activate <환경명>
source deactivate
```