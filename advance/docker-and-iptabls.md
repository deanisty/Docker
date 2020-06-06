#### 起因

有一天，为了研究mongo，在我的腾讯云服务器上部署了docker容器，开启了一个实验用的mongodb容器，命令大概是这样的

```shell
docker run -it -d --name mongo -p 27017:27017 mongo
```
启动了之后大概是这样的

```SHELL
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
b0138c3ffbc8        mongo               "docker-entrypoint.s…"   About an hour ago   Up 3 seconds        0.0.0.0:27017->27017/tcp   mongo
7c16b546870b        golang              "bash"                   3 days ago          Up 3 days           0.0.0.0:80->80/tcp         go
52b0b1e3f881        php                 "docker-php-entrypoi…"   4 days ago          Up 4 days                                      php
```

本来都挺美好，不过，第二天晚上，当我用工具远程连到mongo容器的时候，发现我测试用的库都没了，只剩下一个 warning 库，这个库里有一个readme表，大概这样
(事发当时我没截图，从网上找了一个类似的案例)

![mongodb](/advance/mongodb.jpg)

其实，我的云服务器是开启了

#### Docker和iptables

> 翻译至官方文档：https://docs.docker.com/network/iptables/
