```
$ docker run --name jenkins \
  -d -p 62000:8080 \
  -u root \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  ghcr.io/subicura/jenkins:lts
$ docker logs jenkins # 비밀번호 확인
```



젠킨스로 접속하여 '새 작업을 하고

구성에서



```
node {
    def IMAGE_TAG="v1"

  stage('Deploy') {
      sh """
          set +e

​          docker stop echo_1
​          docker rm echo_1
​          docker run -d --network=exam2-network --name echo_1 -e VIRTUAL_HOST=echo.3.34.94.118.nip.io hello-world:${IMAGE_TAG}

​          sleep 5

​          docker stop echo_2
​          docker rm echo_2
​          docker run -d --network=exam2-network --name echo_1 -e VIRTUAL_HOST=echo.3.34.94.118.nip.io hello-world:${IMAGE_TAG}

​          set -e
​      """
  }

}
```

sh """ 이 안에 코드를 작성한다 docker를 대신하여 일을 하는 jenkins """

v1 이미지와 v2 이미지를 js 파일을 이름을 달리해서 build한 후

http://echo.3.34.94.118.nip.io:64000/ 로 들어가면 v1 v2마다 다른 결과 값을 확인 가능



