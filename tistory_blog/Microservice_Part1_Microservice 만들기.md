> 마이크로 서비스 실습을 위해 시리즈로 Posting할 예정입니다. 
>  
> freeCodeCamp의 [Learn Kubernetes in Under 3 Hours: A Detailed Guide to Orchestrating Containers](https://medium.freecodecamp.org/learn-kubernetes-in-under-3-hours-a-detailed-guide-to-orchestrating-containers-114ff420e882) 블로그 글을 토대로 작성하였습니다.
>
> 제 코드는 [여기서](https://github.com/wooyoung85/kuber-sample-apps) 확인하실 수 있습니다.


# 1. Python Application
> 파이썬의 TextBlob 패키지를 활용하여 문장의 감정을 분석하는 간단한 Application

## 가상환경 구성
#### 1. [pyenv](https://github.com/pyenv/pyenv)와 [pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv)를 사용하여 python 환경 구성

#### 2. `pyenv activate <가상환경명>` 으로 가상환경을 활성화 한 후 아래 내용을 진행

```shell
# 가상환경 생성
$ pyenv virtualenv 3.6.4 <가상환경명> 

# 가상환경 활성화
$ pyenv activate <가상환경명>

# 가상환경 비활성화
$ pyenv deactivate
```

## django를 활용하여 rest api 만들기

#### 1. django 관련 패키지 설치

```shell
$ pip install django djangorestframework
```

#### 2. 프로젝트 생성
```shell
$ mkdir kuber-sample-apps
$ cd kuber-sample-apps
$ django-admin startproject django_app
```

```
# 프로젝트 구조
kuber-sample-apps
└─── django_app
    └─── django_app
    └─── manage.py
```

#### 3. analysis app 생성
```shell
$ cd django_app
$ python manage.py startapp analysis
```

```
# 프로젝트 구조
kuber-sample-apps
└─── django_app
    └─── analysis
    └─── django_app
    └─── manage.py
```

#### 4. settings.py 설정
```python
ALLOWED_HOSTS = ['*'] 
INSTALLED_APPS = [ 
    ... 
    'rest_framework', 
    'analysis.apps.AnalysisConfig', 
]
```

#### 5. djangorestframework를 활용한 rest api 만들기
- analysis/model.py
    ```python
    # Analysis 모델 생성
    from django.db import models 

    class Analysis(models.Model): 
        sentence = models.CharField(max_length=200)
        polarity = models.CharField(max_length=10)
    ```

- 마이그레이션
    ```shell
    $ python manage.py makemigrations 
    $ python manage.py migrate
    ```

- analysis/serializer.py
    ```python
    from rest_framework import serializers 
    from .models import Analysis 

    class AnalysisSerializer(serializers.ModelSerializer): 
        class Meta: 
            model = Analysis # 모델 설정 
            fields = ('sentence','polarity') # 필드 설정
    ```

- analysis/views.py  
(viewset을 사용하면 CRUD 가 가능한 restful api가 생생됨)
    ```python
    from rest_framework import viewsets 
    from .serializers import AnalysisSerializer 
    from .models import Analysis 

    class AnalysisViewSet(viewsets.ModelViewSet): 
        queryset = Analysis.objects.all() 
        serializer_class = AnalysisSerializer
    ```

- django_app/urls.py
    ```python
    from django.conf.urls import url, include
    from django.contrib import admin
    from rest_framework import routers
    from analysis.views import AnalysisViewSet

    router = routers.DefaultRouter()
    router.register('analysis', AnalysisViewSet)

    urlpatterns = [
        url(r'^admin/', admin.site.urls),
        url(r'^', include(router.urls)), 
    ]
    ```

#### 6. 실행
```shell
$ python manage.py runserver 
```
#### 7. http://localhost:8000 접속해서 확인

## Celery 연동
#### 1. Redis 설치
    ```shell
    $ brew install redis
    $ redis-server
    ```
#### 2. 관련 패키지 설치
    ```shell
    $ pip install 'celery[redis]'
    $ pip install textblob
    $ python -m textblob.download_corpora
    ```
#### 3. settings.py 설정 추가
    ```python
    # REDIS
    REDIS_URL = "redis://{host}:{port}".format(
        host=os.getenv('REDIS_HOST', 'localhost'),
        port=os.getenv('REDIS_PORT', '6379')
    )

    # Celery application definition
    CELERY_BROKER_URL = REDIS_URL
    CELERY_RESULT_BACKEND = REDIS_URL
    CELERY_ACCEPT_CONTENT = ['application/json']
    CELERY_RESULT_SERIALIZER = 'json'
    CELERY_TASK_SERIALIZER = 'json'
    ```
#### 4. celery.py 추가
    ```python
    # django_app/celery.py

    from __future__ import absolute_import, unicode_literals
    import os
    from celery import Celery

    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'django_app.settings')
    app = Celery('django_app')
    app.config_from_object('django.conf:settings', namespace='CELERY')

    app.autodiscover_tasks()

    @app.task(bind=True)
    def debug_task(self):
        print('Request: {0!r}'.format(self.request))
    ```
#### 5. tasks.py 만들기 (Text 감정 분석 Task)
    ```python
    # analysis/tasks.py

    from __future__ import absolute_import, unicode_literals
    from django_app.celery import app
    from celery import task
    from textblob import TextBlob

    @app.task
    def sentence_analysis(sentence):
        polarity = TextBlob(sentence).sentences[0].polarity
        return polarity
    ```
#### 6. Celery 실행
    Redis가 실행된 상태에서 실행해야 함 (`ps -ef | grep redis` 으로 확인 가능)
    ```shell
    $ celery -A django_app worker -l info

    celery@uyeong-ui-MacBook-Pro.local v4.2.1 (windowlicker)

    Darwin-17.7.0-x86_64-i386-64bit 2018-09-27 09:54:31

    [config]
    .> app:         django_app:0x10ae6b8d0
    .> transport:   redis://localhost:6379//
    .> results:     redis://localhost:6379/
    .> concurrency: 12 (prefork)
    .> task events: OFF (enable -E to monitor tasks in this worker)

    [queues]
    .> celery           exchange=celery(direct) key=celery


    [tasks]
    . analysis.tasks.sentence_analysis
    . django_app.celery.debug_task

    [2018-09-27 09:54:31,924: INFO/MainProcess] Connected to redis://localhost:6379//
    [2018-09-27 09:54:31,935: INFO/MainProcess] mingle: searching for neighbors
    [2018-09-27 09:54:32,953: INFO/MainProcess] mingle: all alone
    [2018-09-27 09:54:32,965: WARNING/MainProcess] /Users/wooyoung/.pyenv/versions/3.6.4/envs/kuber/lib/python3.6/site-packages/celery/fixups/django.py:200: UserWarning: Using settings.DEBUG leads to a memory leak, never use this setting in production environments!
    warnings.warn('Using settings.DEBUG leads to a memory leak, never '
    [2018-09-27 09:54:32,965: INFO/MainProcess] celery@uyeong-ui-MacBook-Pro.local ready.
    ```

#### 7. 확인
    ```shell
    $ python manage.py shell
    Python 3.6.4 (default, Sep  3 2018, 13:18:55)
    [GCC 4.2.1 Compatible Apple LLVM 9.1.0 (clang-902.0.39.2)] on darwin
    Type "help", "copyright", "credits" or "license" for more information.
    (InteractiveConsole)
    >>> from django_app.celery import debug_task
    >>> debug_task.delay()
    <AsyncResult: 1d2eebe6-320d-459f-a31e-1439810f3fd6>

    >>> from analysis.tasks import sentence_analysis
    >>> result = sentence_analysis.delay('happy coding')
    >>> result.get()
    0.8
    ```


## text 감정 분석 api 만들기
#### 1. api app 만들기 (Text 감정 분석을 위해 외부 Application에서 호출하게 될 api)
    ```shell
    $ python manage.py startapp api
    ```

    ```
    # 프로젝트 구조
    kuber-sample-apps
    └─── django_app
        └─── analysis
        └─── api
        └─── django_app
        └─── manage.py
    ```

    ```python
    # django_app/settings.py에 추가
    INSTALLED_APPS = [ 
        ... 
        'api.apps.ApiConfig', 
    ]
    ```

    
#### 2. api 경로 설정
    http://localhost:8000/api/analysis?sentence=<문장> 이런 식으로 api 경로를 구성

- django_app/urls.py에 추가
    ```python
    urlpatterns = [
        ...
        url(r'^api/', include('api.urls')),
    ]
    ```
- api/urls.py
    ```python
    from django.conf.urls import include, url
    from . import views

    app_name = 'api'
    urlpatterns = [
        url(r'^analysis$', views.analysis, name='request-analysis'),
    ]
    ```

- api/views.py
    ```python
    from analysis.serializers import AnalysisSerializer
    from django.http import JsonResponse
    from analysis.tasks import sentence_analysis

    def analysis(request):
        data = {}
        sentence = request.GET['sentence']
        task = sentence_analysis.delay(sentence)
        polarity = task.wait(timeout=None, propagate=True)
        data['sentence'] = sentence
        data['polarity'] = polarity
        
        serializer = AnalysisSerializer(data=data)
        if serializer.is_valid():
            serializer.save()
        return JsonResponse(data)
    ```
#### 3. http://localhost:8000/api/analysis?sentence=<문장> 호출하여 확인


# 2. Spring Boot Application
> 클라이언트(React Application)의 요청을 받아서 Python Application으로 전달해주는 역활
## 환경세팅
JDK8과 메이븐이 설치되어 있어야 함

## 프로젝트 생성
#### 1. https://start.spring.io/ 에서 Generate Project 후 압축파일 다운로드
    - Group : com.sa.web
    - Artifact : sentiment-analysis
    - Dependencies : Jersey, Web
#### 2. 다운 받은 압축파일 해제한 후 kuber-sample-apps 폴더 밑으로 이동
    ```
    # 프로젝트 구조
    kuber-sample-apps
    └─── django_app
    └─── springboot_app
        └─── src
        └─── mvnw
        └─── mvnw.cmd
        └─── pom.xml
    ```

## 코드 추가
- `src > main > java > com > sa > web` 밑에 `dto` 폴더 생성
- SentenceDto.java
    ```java
    package com.sa.web.dto;

    public class SentenceDto {
        private String sentence;

        public SentenceDto() {
        }

        public String getSentence() {
            return sentence;
        }

        public void setSentence(String sentence) {
            this.sentence = sentence;
        }

        @Override
        public String toString() {
            return sentence;
        }
    }
    ```
- SentimentDto.java
    ```java
    package com.sa.web.dto;

    public class SentimentDto {

        private String sentence;
        private float polarity;

        public SentimentDto() {
        }

        public SentimentDto(String sentence, float polarity) {
            this.sentence = sentence;
            this.polarity = polarity;
        }

        public String getSentence() {

            return sentence;
        }

        public void setSentence(String sentence) {
            this.sentence = sentence;
        }

        public float getPolarity() {
            return polarity;
        }

        public void setPolarity(float polarity) {
            this.polarity = polarity;
        }

        @Override
        public String toString() {
            return "SentimentDto{" +
                    "sentence='" + sentence + '\'' +
                    ", polarity=" + polarity +
                    '}';
        }
    }
    ```

- SentimentController.java
    ```java
    package com.sa.web;

    import com.sa.web.dto.SentenceDto;
    import com.sa.web.dto.SentimentDto;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.web.bind.annotation.*;
    import org.springframework.web.client.RestTemplate;

    @CrossOrigin(origins = "*")
    @RestController
    public class SentimentController {

        @Value("${django_app.url}")
        private String saLogicApiUrl;

        @PostMapping("/sentiment")
        public SentimentDto sentimentAnalysis(@RequestBody SentenceDto sentenceDto) {
            RestTemplate restTemplate = new RestTemplate();

            return restTemplate.getForObject(saLogicApiUrl + "/api/analysis?sentence="+ sentenceDto.getSentence(), SentimentDto.class);
        }

        @GetMapping("/testHealth")
        public String testHealth() {
            return "Health Check";
        }
    }
    ```

## Jar파일로 패키징
```shell
$ mvn install
```

## 실행
target 폴더로 이동 후 패키징 된 jar파일 실행  
(실행 시 Command line arguments를 통해 django_app 의 url 전달)
```shell
$ cd target
$ java -jar sentiment-analysis-0.0.1-SNAPSHOT.jar --django_app.url=http://127.0.0.1:8000
```

# 3. React Application
> 사용자가 입력한 문장을 Spring Boot Application 쪽으로 전달

## 프로젝트 생성
https://github.com/facebook/create-react-app 참조

```shell
$ npx create-react-app react_app
$ cd react_app
```
```
# 프로젝트 구조
kuber-sample-apps
└─── django_app
└─── springboot_app
└─── react_app
    └─── node_modules
    └─── package.json
    └─── public
    └─── src
    └─── yarn.lock
```

## 패키지 추가
- package.js 에 아래 코드 추가
    ```json
    "dependencies": {
        ...
        "material-ui": "^0.20.0",
        "prop-types": "latest"
    }
    ```
    ```shell
    $ npm install
    ```

## 코드 추가
- `src` 폴더 밑에 `components` 폴더 추가
- `components` 폴더에 `Polarity.js` 생성
    ```javascript
    import React, {Component} from 'react';
    import PropTypes from 'prop-types';

    class Polarity extends Component {

        propTypes = {
            sentence: PropTypes.string.isRequired,
            polarity: PropTypes.number.isRequired
        };

        render() {
            const green = Math.round((this.props.polarity + 1) * 128);
            const red = 255 - green;
            const textColor = {
                backgroundColor: 'rgb(' + red + ', ' + green + ', 0)',
                padding: '15px'
            };

            return <div style={textColor}>"{this.props.sentence}" has polarity of {this.props.polarity} </div>
        }
    }

    export default Polarity;
    ```

- `App.js` 파일 수정
    ```javascript
    import React, {Component} from 'react';
    import './App.css';
    import MuiThemeProvider from 'material-ui/styles/MuiThemeProvider';
    import TextField from 'material-ui/TextField';
    import RaisedButton from 'material-ui/RaisedButton';
    import Paper from 'material-ui/Paper';
    import Polarity from "./components/Polarity";

    const style = {
        marginLeft: 12,
    };

    class App extends Component {
        constructor(props) {
            super(props);
            this.state = {
                sentence: '',
                polarity: undefined
            };
        };

        analyzeSentence() {
            fetch('http://localhost:8080/sentiment', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({sentence: this.textField.getValue()})
            })
                .then(response => response.json())
                .then(data => this.setState(data));
        }

        onEnterPress = e => {
            if (e.key === 'Enter') {
                this.analyzeSentence();
            }
        };

        render() {
            const polarityComponent = this.state.polarity !== undefined ?
                <Polarity sentence={this.state.sentence} polarity={this.state.polarity}/> :
                null;

            return (
                <MuiThemeProvider>
                    <div className="centerize">
                        <Paper zDepth={1} className="content">
                            <h2>Sentiment Analyser</h2>
                            <TextField ref={ref => this.textField = ref} onKeyUp={this.onEnterPress.bind(this)}
                                    hintText="Type your sentence."/>
                            <RaisedButton  label="Send" style={style} onClick={this.analyzeSentence.bind(this)}/>
                            {polarityComponent}
                        </Paper>
                    </div>
                </MuiThemeProvider>
            );
        }
    }

    export default App;
    ```

- `App.css`에 아래 코드 추가
    ```css
    ...
    .centerize {
        width: 400px;
        height: 100px;

        position: absolute;
        top:0;
        bottom: 0;
        left: 0;
        right: 0;
        margin: auto;
    }

    .content {
        display: inline-block;
        padding: 20px;
    }

    h2{
        text-align: center;
    }
    ```
## 실행
```shell
$ npm start
```

## 참고자료
[Learn Kubernetes in Under 3 Hours: A Detailed Guide to Orchestrating Containers](https://medium.freecodecamp.org/learn-kubernetes-in-under-3-hours-a-detailed-guide-to-orchestrating-containers-114ff420e882)  
[Using Django 2 with Celery and Redis](https://medium.com/@markgituma/using-django-2-with-celery-and-redis-21343284827c)  
[Kubernetes, Local to Production with Django: 4 - Celery with Redis and Flower](https://medium.com/@markgituma/kubernetes-local-to-production-with-django-4-celery-with-redis-and-flower-df48ab9896b7)
[django REST framework로 간단한 api 만들기](http://jamanbbo.tistory.com/43)