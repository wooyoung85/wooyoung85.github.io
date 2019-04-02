오래 걸리는 python 프로그램을 병렬로 처리하는 방법에 대해 정리하였습니다.

# Celery 란?
> Celery는 분산 메세지 전달에 기반한 비동기 작업 큐

별도로 실행 중인 Worker Process가 Broker로부터 Message를 전달 받아 작업을 대신 수행해 주는 라이브러리입니다.

# Celery를 활용한 병렬 작업 처리
Celery 4.0 이상 버전은 Windows 환경을 공식 지원하지 않기 때문에 실습은 CentOS 7에서 진행하겠습니다.  
CentOS 환경 구성은 [Virtual Box를 활용한 Spark 실습 환경 구축](https://wooyoung85.tistory.com/35) Post를 참고하시기 바랍니다~

## Python 설치
```bash
# 필요한 패키지 설치
yum update
yum install yum-utils
yum groupinstall development
yum install zlib-devel bzip2 bzip2-devel readline-devel sqlite sqlite-devel openssl-devel xz xz-devel libffi-devel findutils

# pyenv 설치
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bash_profile
source ~/.bash_profile

# pyenv virtualenv 설치
git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bash_profile
source ~/.bash_profile

# 가상환경 만들기
pyenv install 3.6.4
pyenv virtualenv 3.6.4 celery
pyenv activate celery
```

# Python Package 설치
```bash
(celery) $> pip install celery[redis]
```

## Broker 설치
Celery 공식문서에선 RabbitMQ를 추천하고 있지만 편의상 Redis로 진행하겠습니다.

```bash
# epel 저장소 설치
rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
# redis 설치
yum --enablerepo=epel install redis

# Redis 서비스 실행 및 확인
systemctl start redis
ps -ef | grep redis
```

## Python 프로그래밍
- `home` 폴더 밑에 작업 폴더 생성 후 이동
    ```bash
    mkdir workspace && cd workspace
    ```
- `tasks.py` 파일 생성 (`vi tasks.py`) 후 아래 내용 입력
    ```python
    import time, datetime
    from celery import Celery, group

    app = Celery('tasks', backend='redis://localhost:6379', broker='redis://localhost:6379')

    @app.task
    def add_test1(limit):
        print(datetime.datetime.now())
        result = [add(n,n) for n in range(1, limit)]
        print(datetime.datetime.now())
        return result

    @app.task
    def add_test2(limit):
        print(datetime.datetime.now())
        job = group([add.s(n,n) for n in range(1, limit)])
        result = job.apply_async()

        while(result.successful() != True):
            pass

        print(datetime.datetime.now())
        return result

    @app.task
    def add(x, y):
        time.sleep(10)
        return x + y
    ```

## Celery 실행
```bash
celery -A tasks worker --loglevel=INFO --autoscale=10,3
```
--autoscale : 자동으로 worker를 추가 및 삭제하는 설정  

**위와 같이 설정하면 최대 10개까지 늘어나고 최소 3개의 worker를 유지하게 됨**

## 참고자료
[Next Steps-Celery 4.2.0 documentation](http://docs.celeryproject.org/en/latest/getting-started/next-steps.html#canvas-designing-work-flows)  
[celery.worker.autoscale-Celery 4.2.0 documentation](http://docs.celeryproject.org/en/latest/internals/reference/celery.worker.autoscale.html)
