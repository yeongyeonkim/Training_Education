# 도커 컨테이너 만들기

**2020/9/21**

실습환경 : 3.34.94.118:4200 ip로 들어가서

ubuntu / apple-kiwi-apple로 Linux 환경으로 접속

`$ curl -s https://get.docker.com/ | sudo sh`

`$ sudo usermod -aG docker $USER`

서버 재접속

`docker version`



# Task 1. 컨테이너 실행하기 : run

**ubuntu 18.04 sh 실행하기**

- -i와 -t 옵션을 이용하여 컨테이너 접속

`$ docker run --rm -it ubuntu:18.04 /bin/sh`

**CentOS7 실행하기** 

- 현재 리눅스 배포판(ubuntu)과 다른 종류의 배포판(centos)을 생성

`$ docker run --rm -it centos:7 /bin/sh`

`sh-4.2# cat /etc/redhat-release`

**샘플 웹 애플리케이션(v1) 실행하기**

- -d 옵션을 이용하여 백그라운드 컨테이너로 실행하고 -p 옵션을 이용하여 호스트의 포트와 컨테이너의 포트를 연결

`-p {호스트 포트} : {컨테이너 포트}`

`$ docker run -d -p 60000:4567 raccoony/docker-workshop-app:1`

- http://{IP}:60000 접속 확인 (여기서 IP는 쉘로 들어갈때의 IP http://3.34.94.118:60000)

**샘플 웹 애플리케이션(v2) 실행하기**

- -e 옵션을 이용하여 환경변수를 지정

`$ docker run -d \
 -p 60001:4567 \
 -e ENDPOINT=https://workshop-docker-kr.herokuapp.com/ \
 -e PARAM_NAME=haha \
 raccoony/docker-workshop-app:2`

* http://IP:60001 접속하면 웹 게시판에 글을 자동으로 작성 
* PARAM_NAME의 값을 바꿔서 실행하면 https://workshop-docker-kr.herokuapp.com/ 게시판 확인

**샘플 웹 애플리케이션(v3) 실행하기**

`$ docker run -d \
 -p 60001:4567 \
 -e ENDPOINT=https://workshop-docker-kr.herokuapp.com/ \
 -e PARAM_NAME=haha \
-e PARAM_MESSAGE=하하하 \
 raccoony/docker-workshop-app:2`



**Redis 실행하기**

`$ docker run --name=redis -d -p 61000:6379 redis`
`$ telnet localhost 61000`
`set docker container`
`get docker`
`keys *`
`quit`

다른 포트로 2번째 레디스 서버 생성

`$ docker run --name=redis2 -d -p 61001:6379 redis`
`$ telnet localhost 61001`
`get docker`
`dbsize`
`quit`



**MySQL 실행하기**

- https://hub.docker.com/_/mysql 에서 어떤 환경변수를 사용하는지 확인

`$ docker run -d -p 3306:3306 \`
`-e MYSQL_ALLOW_EMPTY_PASSWORD=true \`
`-e MYSQL_DATABASE=wp \`
`-e MYSQL_USER=wp \`
`-e MYSQL_PASSWORD=wp \`
`--name mysql \`
`mysql:5.7`



**Wordpress 실행하기**

* https://hub.docker.com/_/wordpress 에서 어떤 환경변수를 사용하는지 확인. 앞에서 실행한 MySQL을 이용함

`$ docker run -d -p 62000:80 \`
`-e WORDPRESS_DB_HOST=172.17.0.1 \`
`-e WORDPRESS_DB_NAME=wp \`
`-e WORDPRESS_DB_USER=wp \`
`-e WORDPRESS_DB_PASSWORD=wp \`
`wordpress`

* http://{IP}:62000 접속하기

---

# Task 2. 컨테이너 목록 : ps

`docker ps`

`docker ps -a` - 중지된 컨테이너도 확인

---

# Task 3. 컨테이너 명령어 실행 : exec

`$ docker exec {CONTAINER_ID} {명령어}`

**컨테이너의 bash 쉘에 접속하기**

`$ docker exec -it CONTAINER_ID bash`

**컨테이너 프로세스 목록 보기**

`$ docker exec CONTAINER_ID ps aux`

**MySQL 접속**

`$ docker exec -it mysql mysql -u wp -p`

### # docker run/exec 차이점 ★

- run - 새로 container를 만들어서 실행.
- exec - 실행 중인 container에 명령어 전달.

----

# Task 4. 컨테이너 중지 : stop

`$ docker stop {CONTAINER_ID} [ {CONTAINER_ID}...]`

---

# Task 5. 컨테이너 제거 : rm

`$ docker rm {CONTAINER_ID} [ {CONTAINER_ID}...]`

- stop을 포함한 강제 종료는 `rm -f`

---

# Task 6. 컨테이너 로그 : logs

`$ docker logs {CONTAINER_ID}
$ docker logs -f {CONTAINER_ID}
$ docker logs --tail 20 {CONTAINER_ID}`

---

# Task 7. 이미지 목록 : images

`$ docker images`

---

# Task 8. 이미지 다운로드 : pull

`$ docker pull {IMAGE_NAME:IMAGE_TAG}`

---

# Task 9. 이미지 제거 : rmi

`$ docker rmi {IMAGE_NAME:IMAGE_TAG}`

---

# Task 10. 네트워크 생성 : network create

같은 네트워크에 속한 컨테이너는 서로의 이름을 도메인으로 사용할 수 있음.

`$ docker network create app-network`

네트워크에 연결된 컨테이너 생성

`$ docker run -d \ `
`-e MYSQL_ALLOW_EMPTY_PASSWORD=true \ `
`-e MYSQL_DATABASE=wp \ `
`-e MYSQL_USER=wp \ `
`-e MYSQL_PASSWORD=wp \ `
`--name mysql \ `
`--network=app-network \ `
`mysql:5.7`



`$ docker run -d -p 63000:80 \
 --network=app-network \
 -e WORDPRESS_DB_HOST=mysql \
 -e WORDPRESS_DB_NAME=wp \
 -e WORDPRESS_DB_USER=wp \
 -e WORDPRESS_DB_PASSWORD=wp \
 wordpress`

---

# Task 11. -v 옵션으로 디렉토리 연결

컨테이너는 삭제하면 내부에 생성된 데이터도 같이 삭제되기 때문에 저장이 필요한 데이터는 반드시 호스트의 파일시스템에 데이터를 저장해야함



**Ghost 블로그 실행**

`$ docker run -d -it --name blog --rm -p 64000:2368 ghost`

http://{IP}:64000/ghost 접속 후 글을 작성

컨테이너 제거하고 다시 실행하기

`$ docker stop blog`

`$ docker run -d --rm -p 64000:2368 --name blog ghost`

이전에 설정한 내용이 모두 사라짐! 컨테이너 특정 디렉토리를 호스트 디렉토리와 연결

`$ mkdir content
$ docker run -d --rm -p 64001:2368 --name blog -v $(pwd)/content:/var/lib/ghost/content ghost`

http://IP:64001/ghost 접속 후 글을 작성

컨테이너 제거하고 다시 실행하기

---

# Exam 1. nginx 컨테이너 만들기

![](C:\Users\yeongyeon.kim\Desktop\nginx.PNG)

1. `docker run -p 50000:80 -d nginx:latest`

mkdir content

cd content

nano (해서 index.html을 만듬)

1. `docker run -p 3001:80 -d -v $(pwd)/content:/usr/share/nginx/html nginx:latest`

pwd - 현재 디렉토리

---

# Exam 2. php 컨테이너 실행하기



![](C:\Users\yeongyeon.kim\Desktop\php.PNG)

`mkdir php`

`cd php`

`nano` 해서 소스코드를 가진 hello.php를 만든다.



`docker run -it --name php -v /home/ubuntu/php:/usr/src/myapp -w /usr/src/myapp php:7 php hello.php`

-it : 터미널 입력을 위한 옵션

`docker run -it -v $(pwd)/php:/tmp php:7`

`tmp 임시 폴더를 많이 사용함`

`docker run -it -v $(pwd)/php:/tmp php:7 /bin/bash (배쉬버전)`

`docker run -it -v $(pwd)/php:/tmp php:7 php /tmp/hello.php (바로실행시켜보는거)`

`docker system prune -a` (현재 사용하지 않는 컨테이너와 이미지를 전부 제거합니다.)



`docker stop $(docker ps -a -q)`

`docker rm $(docker ps -a -q)`

도커 컨테이너 전부 삭제



