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

其实，我的云服务器是开启了iptables的，按理说如果黑客不攻破我主机的登陆用户名和密码是不可能侵入我的容器的，于是我检查了一下iptables列表，不幸的是我发现了很多docker添加的规则，其中有一条就是开放了 27017 端口给任何来源：

```SHELL
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED
ACCEPT     all  --  anywhere             anywhere
ACCEPT     icmp --  anywhere             anywhere
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:http
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:tr-rsrb-p3
REJECT     all  --  anywhere             anywhere             reject-with icmp-port-unreachable

Chain FORWARD (policy DROP)
target     prot opt source               destination
DOCKER-USER  all  --  anywhere             anywhere
DOCKER-ISOLATION-STAGE-1  all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED
DOCKER     all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Chain DOCKER (1 references)
target     prot opt source               destination
ACCEPT     tcp  --  anywhere             172.17.0.4           tcp dpt:http
ACCEPT     tcp  --  anywhere             172.17.0.2           tcp dpt:27017

Chain DOCKER-ISOLATION-STAGE-1 (1 references)
target     prot opt source               destination
DOCKER-ISOLATION-STAGE-2  all  --  anywhere             anywhere
RETURN     all  --  anywhere             anywhere

Chain DOCKER-ISOLATION-STAGE-2 (1 references)
target     prot opt source               destination
DROP       all  --  anywhere             anywhere
RETURN     all  --  anywhere             anywhere

Chain DOCKER-USER (1 references)
target     prot opt source               destination
RETURN     all  --  anywhere             anywhere
```

可算找到原因了，其实只要docker开启了主机和容器的端口映射，它就会在防火墙规则表里增加一条ACCEPT规则，所以防火墙其实对于docker来说没有任何用处，所有
容器都暴露到外网环境了。

一气之下，我找到了docker官方文档，研究了一下，下面是译文！


#### Docker和iptables

> 翻译至官方文档：https://docs.docker.com/network/iptables/

