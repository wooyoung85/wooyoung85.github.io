# Script 내용
- Django로 개발한 웹 어플리케이션을 Gunicorn을 활용하여 웹서버를 구동하는 상황에서 코드 변경이나 새로 작성한 파일을 반영하는 스크립트
- pyenv로 python 가상환경을 사용함

# 기존 Script
- `git pull` 하는 과정에서 입력 정보가 잘못 됐을 경우 엉망징창이 됨  
(직접 `git pull`과 `collectstatic` 작업을 수행한 후 `Gunicorn`을 시작해야 함)

    ```bash
    echo "pyenv 활성화" && 
    source /home/wooyoung/.pyenv/versions/3.6.4/envs/venv/bin/activate && 
    echo "Gunicorn 중지" &&  
    pkill gunicorn && 
    echo "Git Pull" && 
    git pull && 
    echo "Collecting statistic" && 
    yes "yes" | python manage.py collectstatic  && 
    echo "Gunicorn 시작" && 
    gunicorn --bind 0.0.0.0:8000 conf.wsgi:application && 
    echo "Update 완료!"
    ```

# 개선 스크립트
- `git pull` 하는 과정에서 입력 정보가 잘못되면 스크립트를 다시 실행하면 됨
    ```bash
    echo "Git Pull"
    if ! (git pull) then
        echo "Git Pull 실패"
        exit 1        
    else
        echo "Git Pull 성공"
        echo "pyenv 활성화" && 
        source /home/wooyoung/.pyenv/versions/3.6.4/envs/venv/bin/activate &&
        echo "Gunicorn 중지" &&  
        pkill gunicorn && 
        echo "Collecting statistic" && 
        yes "yes" | python manage.py collectstatic  && 
        echo "Gunicorn 시작" && 
        gunicorn --bind 0.0.0.0:8000 conf.wsgi:application && 
        echo "Update 완료!"
    fi
    ```