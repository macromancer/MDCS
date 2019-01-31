# MDCS 설치
Ubuntu에 Docker로 설치한다.

## Python 설치
### Anaconda 설치 
```
$ cd ~/Downloads
$ wget https://repo.continuum.io/archive/Anaconda2-2018.12-Linux-x86_64.sh
$ chmod 775 Anaconda2-2018.12-Linux-x86_64.sh
$ ./Anaconda2-2018.12-Linux-x86_64.sh
```
엔터 &rarr; 끝까지 스페이스 바 &rarr; yes &rarr; 엔터 &rarr; yes (PATH 추가. 디폴트가 no이니 꼭 yes 할 것) &rarr; no

다음 명령으로 path가 설정되었는지 확인한다.
```
$ source ~/.bashrc
$ echo $PATH
```
anaconda 실행 폴더 경로 (`/home/사용자아이디/anaconda2/bin`) 가 추가 되었는지 확인
  
안 됐으면 `~/.bashrc` 에 아래 한 줄 추가
```
$ vi ~/.bashrc

...
export PATH=/home/사용자아이디/anaconda2/bin:$PATH
source ~/.bashrc
echo $PATH
```

버전 정보에 Anaconda 어쩌고가 포함되었는지 확인
```
$ python -V
```

### env 생성 (2.7.2)
```
$ conda create -n curator python=2.7
```

### Git 설치
```
$ sudo apt-get install git
```

### 프로젝트 clone
```
$ cd ~/Projects
$ git clone https://github.com/macromancer/MDCS.git
$ cd MDCS
```

## MongoDB 설치
* 참고: https://github.com/usnistgov/MDCS/blob/stable/docs/MongoDB%20Configuration.md
* 참고: https://hub.docker.com/_/mongo/
```
$ cd ~/Projects/MDCS
$ vi conf/mongodb.conf
net:
bindIp: 127.0.0.1
security:
authorization: enabled
storage:
dbPath: /data/db

$ mkdir -p /data/db/mongo
$ docker run --name mongo_mdcs -p 27017:27017 \
-v /data/db/mongo:/data/db \
-v ~/Projects/MDCS/conf:/etc/mongo \
-e MONGO_INITDB_ROOT_USERNAME=admin \
-e MONGO_INITDB_ROOT_PASSWORD='관리자암호' \
--restart unless-stopped \
-d mongo \
--config /etc/mongo/mongodb.conf
```

* 실행 확인
  ```
  $ docker ps
  ```

### MongoDB 설정
* 참고: https://github.com/usnistgov/MDCS/blob/stable/docs/MongoDB%20Configuration.md
* 참고: https://hub.docker.com/_/mongo/  
```
$ docker exec -it mongo_mdcs /bin/bash
# mongo --port 27017  -u "admin" -p "관리자암호" --authenticationDatabase admin
> use mgi
> db.createUser(
{
user: "mgi_user",
pwd: "일반암호",
roles: ["readWrite"]
}
)
> exit
# exit
```

### MDCS setting.py 수정
```
$ cd ~/Projects/MDCS
$ vi mdcs/settings.py
...
# Replace by your own values
MONGO_MGI_USER = "mgi_user"
MONGO_MGI_PASSWORD = "일반암호"
...
# SERVER_URI = 'http://127.0.0.1:8000'
SERVER_URI = 'http://<ip 또는 domain>:8000'
...
# ALLOWED_HOST = []
ALLOWED_HOST = [<ip 또는 domain>]
```

## Redis 설치
* 참고: https://hub.docker.com/_/redis/
```
$ docker run --name redis_mdcs \
-p 6379:6379 \
--restart unless-stopped \
-d redis
$ docker exec -it redis_mdcs /bin/bash
# redis-cli
> ping
> exit
# exit
```

## Python 패키지 설치
```
$ cd ~/Projects/MDCS
$ source activate curator
(curator) $ pip install -e git://github.com/MongoEngine/django-mongoengine.git@v0.2.1#egg=django-mongoengine
(curator) $ pip install --no-cache-dir -r requirements.txt
(curator) $ pip install git+https://github.com/macromancer/core_module_excel_uploader_app.git@1.0.0-rc2.1
(curator) $ pip install --no-cache-dir -r requirements.core.txt
```

## 장고 프로젝트 migrate

먼저 셀러리를 실행하자.
```
(curator) $ celery multi start worker1 -A mdcs -Q feeds -l info --pidfile="./celery_worker.pid"
```

```
(curator) $ cd ~/Projects/MDCS
(curator) $ python manage.py migrate auth
(curator) $ python manage.py migrate
(curator) $ python manage.py collectstatic --noinput
(curator) $ sudo apt-get install gettext
(curator) $ python manage.py compilemessages
(curator) $ python manage.py createsuperuser
loading settings at...:mgi.settings
Username (leave blank to use 'xxx'): <mdcs_superuser_username>
Email address: <mdcs_superuser_email>
Password: 
Password (again): 
Superuser created successfully.
```

## 실행
* 참고: http://i5on9i.blogspot.com/2016/07/celery_21.html
```
$ cd ~/Projects/MDCS
$ source activate curator
(curator) $ celery multi start worker1 -A mgi -Q feeds -l info --pidfile="./celery_worker.pid"
(curator) $ python manage.py runserver 0.0.0.0:8000 &
```
* 일반사용자 화면: `http://127.0.0.1:8000/`
* 관리자 화면: `http://127.0.0.1:8000/admin/`
 
## 참고: 서버 사용자 추가 시 할 일
* docker 그룹에 추가
  ```
  $ sudo gpasswd -a 사용자아이디 docker
  ```
* dev 그룹에 추가
  ```
  $ sudo gpasswd -a 사용자아이디 dev
  ```
* 사용자 홈 디렉토리에 /data 폴더 링크 생성
  ```
  $ cd ~
  $ ln -s /data data
  ```

