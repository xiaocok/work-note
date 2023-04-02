
地址：
```text
https://github.com/tobegit3hub/seagull
```

### docker安装
```shell
docker run -d -p 10086:10086 -v /var/run/docker.sock:/var/run/docker.sock tobegit3hub/seagull
```

### docker-compose
```shell
docker-compose up -d.
```
docker-compose.yml
```yaml
version: '2'

services:
  seagull:
    restart: always
    image: tobegit3hub/seagull
    ports:
      - "10086:10086"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

### 多服务器
链接远程其他服务器的docker
```shell
docker -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock -api-enable-cors=true -d
```

### 源码编译安装：
```shell
1. Install golang and setup $GOPATH
2. go get github.com/astaxie/beego
3. go get github.com/tobegit3hub/seagull
4. go build seagull.go
5. sudo ./seagull
```