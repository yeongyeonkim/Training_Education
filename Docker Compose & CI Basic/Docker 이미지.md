# 도커 이미지 만들기

----

# Task 1. Git을 설치한 ubuntu 이미지



`nano Dockerfile`



`FROM ubuntu:latest`
`RUN apt-get update`
`RUN apt-get install -y git` -y 가 yes를 해준다는 건데 미리 설정해줘야함.



**도커 빌드**

`$ docker build -t ubuntu:git-dockerfile .` . 필요!

ubuntu:git-dockerfile로 태그된 이미지가 생성됬다

`$ docker images | grep ubuntu` 로 확인하면

git-dockerfile로 된 것이 하나 생성된 것을 볼 수 있다.



wget, ip, telnet, tree 설치된 ubuntu:tools 이름의 이미지를 만들자

-> mkdir로 만들고 -> 그곳으로 cd -> nano Dockerfile에

FROM ubuntu:latest
RUN apt-get update
RUN apt-get install -y wget
RUN apt-get install -y iproute2
RUN apt-get install -y telnet
RUN apt-get install -y tree

적고 -> docker built -t ubuntu:tools . -> docker images | grep ubuntu 확인

---

# Task 2. Sinatra 애플리케이션 실행하기

Gemfile 

app.rb 

Dockerfile

다음의 파일들을 만듭니다

**Gemfile**

`source 'https://rubygems.org`

`gem 'sinatra'`

**app.rb**

`require 'sinatra'`
`require 'socket'`

`get '/' do`

`Socket.gethostname`

`end`

**Dockerfile**

1. 우분투 설치

`FROM ubuntu:18.04`
`RUN apt-get -y update`

2. 루비설치

`RUN apt-get -y install ruby`
`RUN gem install bundler`

3. 소스 복사

`COPY . /usr/src/app`

4.  Gem 패키지 설치 (실행 디렉토리 설정)

`WORKDIR /usr/src/app`
`RUN bundle install`

5. Sinatra 서버 실행 (Listen 포트 정의)

`EXPOSE 4567`
`CMD bundle exec ruby app.rb -o 0.0.0.0`



이미지 빌드하기

`$ docker build -t app .`

실행하기

`$ docker run --rm -d -p 60004:4567 app`



**Dockerfile(v2)**

- Base 이미지 최적화

`FROM ruby:2.6`
`COPY . /usr/src/app`
`WORKDIR /usr/src/app`
`RUN bundle install`
`EXPOSE 4567`
`CMD bundle exec ruby app.rb -o 0.0.0.0`

**Dockerfile(v3)**

- 캐시 최적화

`FROM ruby:2.6
COPY Gemfile* /usr/src/app/
WORKDIR /usr/src/app
RUN bundle install
COPY . /usr/src/app
EXPOSE 4567
CMD bundle exec ruby app.rb -o 0.0.0.0`

**Dockerfile(v4)**

`FROM ruby:2.6
COPY Gemfile* /usr/src/app/
WORKDIR /usr/src/app
RUN bundle install --no-rdoc --no-ri
COPY . /usr/src/app
EXPOSE 4567
CMD bundle exec ruby app.rb -o 0.0.0.0`

실습중 --no-rdoc가 잘 되지 않음.

---

# Task 3. Python flask app 만들기

 `docker rmi $(docker images -q)` - 도커 이미지 전체 삭제

# Task 4. Spring Boot + Multistage Build

스프링 부트 기본 샘플 코드 다운로드

`curl https://start.spring.io/starter.zip -d dependencies=web,devtools -d bootVersion=2.1.9.RELEASE -o spring-boot-sample.zip`

**Dockerfile**

`FROM adoptopenjdk/openjdk8:alpine` 
`COPY . /app` 
`WORKDIR /app` 
`RUN ./mvnw package` 
`CMD ["java","-Djava.security.egd=file:/dev/./urandom", "-jar","./target /demo-0.0.1-SNAPSHOT.jar"]`

이미지 빌드하기

`$ docker build -t springboot .`

**Dockerfile(v2)**

`FROM maven:3-jdk-8-slim as build` 
`COPY . /app` 
`WORKDIR /app`

`RUN mvn package` 

`FROM adoptopenjdk/openjdk8:alpine` 
`COPY --from=build /app/target/demo-0.0.1-SNAPSHOT.jar /app.jar` 
`CMD ["java","-Djava.security.egd=file:/dev/./urandom", "-jar","/app.jar "]`

* 위의 From에서는 jar 파일을 만들어줌
* 두번쨰 From - alpine 용량이 굉장히 작은 실행파일이 하나인 이미지
* --from=build (위에서 as build) 
* cmd로 웹을 실행시키는 것

- 멀티 스테이지 빌드를 이용하면 도커 이미지 용량(+시간 또한)을 최적화 할 수 있고 단계별로 별도의 캐시를 적용하므로 더 효율적인 빌드를 구성할 수 있습니다.

---

# Task 5. 환경변수

* php의 post_max_size 설정을 8M => 128M으로 변경

**hello.php**

`<?php phpinfo() ?>`

![](C:\Users\yeongyeon.kim\Desktop\task5.PNG)

**run.sh**

`#!/bin/sh` 
`POST_MAX_SIZE=${POST_MAX_SIZE:-"32M"}` 
`sed -e "s/post_max_size = 8M/post_max_size = ${POST_MAX_SIZE}/" -i usr/local/etc/php/php.ini` 
`php /tmp/hello.php`

**Dockerfile**

`FROM php:7`
`RUN mv /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini`
`ADD run.sh /tmp/run.sh`
`RUN chmod +x /tmp/run.sh`
`ADD hello.php /tmp/hello.php`
`CMD ["/tmp/run.sh"]`



`$ docker run --rm -it php-sample | grep post_max_size`
`post_max_size => 32M => 32M`



`$ docker build -t php-sample .`
`$ docker run -- rm -e POST_MAX_SIZE=128M php-sample`



---

### 도커 허브

* 서버에서 생성한 이미지를 다른 서버에서 사용하려면 중앙 저장소에 이미지를 저장해야 합니다. Docker Hub는 도커에서 만든 무료 저장소 입니다.

`$ docker image ls | grep php`

`$ docker login`
`$ docker tag php-sample {ID}/php-sample`
`$ docker push {ID}/php-sample`
`$ docker pull {ID}/php-sample`

* 생성한 이미지를 도커 허브에 저장한다.

![](C:\Users\yeongyeon.kim\Desktop\DockerHub.PNG)

---

# Exam 1. Nginx를 이용한 정적 페이지 서버 만들기

![](C:\Users\yeongyeon.kim\Desktop\Exam1.PNG)

1. 먼저 도커파일을만든다

**Dockerfile**

`FROM nginx:latest`
`COPY . /usr/share/nginx/html`
`WORKDIR /usr/share/nginx/html`
`EXPOSE 80`

**index.html**

`<html>`

 <head>
 <title>도커 이미지 예제</title>
 <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/
>
 </head>
 `<body>Nginx 서버를 도커 이미지로 만들었습니다.`

 `</body>`
`</html>`

`$ docker build -t {이미지이름:태그}` 빌드하고

`$ docker run -p -d {임의의포트번호}:80 {이미지이름:태그}`

`-d` : detached mode (백그라운드 모드 / 리눅스에선 Daemon이 그런 역할을 함.)

`-p` : 호스트와 컨테이너의 포트를 연결 (포워딩) 

* -p {호스트 포트} : {컨테이너 포트}

`-v` : 호스트와 컨테이너의 디렉터리를 연결 (마운트)

