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

## Docker 설치
* 참고: https://docs.docker.com/install/linux/docker-ce/ubuntu/

### sudo 없이 docker 사용
* 참고: http://iamartin-gh.herokuapp.com/ubuntu-16-04-docker-install/

* docker 그룹에 현재 사용자 추가
  ```
  $ sudo usermod -aG docker $USER
  ```
* 로그아웃 후 다시 로그인

## 참고: HDD mount
* 참고: https://m.blog.naver.com/kimmingul/220639741333
* 참고: https://www.phychode.com/sprt/blog/sprtBlogPost.pem?blogSeq=332
### HDD 연결 확인
```
$ sudo fdisk -l

...
Disk /dev/sdb: 3.7 TiB, 4000787030016 bytes, 7814037168 sectors
...
```

### HDD 파티션 생성
```
$ sudo parted /dev/sdb
(parted) mklabel gpt
(parted) unit TB
(parted) mkpart primary 0.00TB 4.00TB
(parted) print
(parted) quit
$ ls /dev/sdb*
$ sudo fdisk -l
$ sudo mkfs.ext4 /dev/sdb1
```

### 마운트
```
$ sudo mkdir /data
$ sudo mount /dev/sdb1 /data
$ df -H
```
### 자동 마운트
* 자동 마운트를 위해 디스크의 uuid(장치의 유일한 식별id - 주의 : 다른곳에 꽂으면 또 바뀐다)를 확인
  ```
  $ ls -l /dev/disk/by-uuid
  ```
* 출력결과 중 sdb1 의 uuid를 찾아 복사
* fstab에 마운트를 걸어줌
  ```
  $ sudo vi /etc/fstab
  ```
* vi 편집화면이 나타나면 맨 아래로 이동해 다음과 같이 삽입
  ```
  ...
  UUID=b7e049ed-b1bf-4a24-a008-76401808ab9f /data ext4 defaults 0 0
  ```
* 마운트 확인
  ```
  $ sudo mount -a
  ```
* 재부팅 후 자동 마운트가 되었는지 확인

### 공용 data 폴더 설정
멤버들이 /data 디렉토리를 공유할 수 있도록 설정한다.
* dev 그룹 생성
  ```
  $ sudo groupadd dev
  $ cat /etc/group
  ```
* kikim1, leehsong 을 dev 그룹에 추가
  ```
  $ sudo gpasswd -a kikim1 dev
  $ cat /etc/group
  $ cat /etc/passwd
  ```
* /data 디렉토리 그룹 owner 변경
  ```
  $ sudo chown -R :dev /data
  $ cd /data
  $ sudo chmod 775 .
  $ sudo setfacl -m default:group:dev:wrx .
  $ sudo chmod g+s . # /data 내에 만들어지는 항목들이 dev 그룹으로 생성되도록 설정
  ```
* 사용자 홈 디렉토리에 심볼릭 링크 생성
  ```
  $ cd ~
  $ ln -s /data data
  ```

* 참고: [그룹 아이디 상속하도록 설정](https://serverfault.com/questions/96338/specify-default-group-and-permissions-for-new-files-in-a-certain-directory)
* 참고: [리눅스 그룹 공유 디렉토리 만들기](http://miruel.tistory.com/entry/Linux-%EA%B7%B8%EB%A3%B9-%EA%B3%B5%EC%9C%A0-%EB%94%94%EB%A0%89%ED%86%A0%EB%A6%AC-%EB%A7%8C%EB%93%A4%EA%B8%B0) (default 설정 명령이 잘못 적혀있음)
* 참고: [리눅스 ACL 설정](https://www.lesstif.com/pages/viewpage.action?pageId=18219571)
* 참고: [리눅스 ACL 명령어](https://m.blog.naver.com/PostView.nhn?blogId=minki0127&logNo=220729799731&proxyReferer=https%3A%2F%2Fwww.google.co.kr%2F)

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
#HOST = '127.0.0.1'
#HOST = '<ip 또는 domain>'
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
* 참고: https://github.com/usnistgov/MDCS/blob/stable/docs/Installation%20Instructions%20for%20Linux.md#install-required-python-packages 
* 참고: https://github.com/usnistgov/MDCS/blob/stable/docs/Required%20Python%20Packages.md 
* pip + `requirements.txt` 대신 conda + `environment.yml` 사용
    * 참고: https://stackoverflow.com/questions/47999477/bash-script-to-conda-install-requirements-txt-with-pip-follow-up
    * [`environment.yml`](https://raw.githubusercontent.com/c51tech/kims/master/environment.yml?token=ADHvX5nJA2StbW80PtGLj7C-dfp5uCQ0ks5cVN-vwA%3D%3D) 을 `~/Projects/MDCS/docs` 로 복사
```
$ source activate curator
(curator) $ conda env update --file docs/environment.yml
```
### django-mongoengine 설치
djanog 1.8 버전을 지원하는 v0.2.1 버전을 설치해야 함 
```
(curator) $ conda install setuptools
(curator) $ cd ~/Projects
(curator) $ git clone -b v0.2.1 https://github.com/MongoEngine/django-mongoengine.git
(curator) $ cd django-mongoengine
(curator) $ python setup.py install
```

## 장고 프로젝트 migrate
```
(curator) $ cd ~/Projects/MDCS
(curator) $ python manage.py migrate
(curator) $ python manage.py createsuperuser
loading settings at...:mgi.settings
Username (leave blank to use 'kikim'): leehsong
Email address: leehsong@gmail.com
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
(curator) $ python manage.py runserver 0.0.0.0:8000 --noreload &
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

