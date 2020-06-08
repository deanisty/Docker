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

本来都挺美好，不过，第二天晚上，当我用工具远程连到mongo容器的时候，发现我测试用的库都没了，只剩下一个名字叫 `README_TO_RECOVER_YOUR_DATA` 的库，
这个库里只有一个readme表，大概长这样

![mongodb](/advance/mongo.PNG)

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

在Linux系统中，Docker通过操作 iptables 规则来提供网络隔离功能。这是一个实现上的细节，一般情况下你不应该修改 Docker 对 iptables 策略新增的规则，
但是如果你想额外再定制规则的话，这可能会有一定的影响。

如果运行 docker 的主机暴露在外网环境下，可能会需要使用 iptables 来阻止未授权的访问，尤其是对于容器中服务的访问。

#### 在 Docker 规则之前添加iptables 规则

Docker 添加了两个自定义的iptables 规则链：DOCKER-USER 和 DOCKER，以此来保证所有入网的流量包都首先通过这两个规则的检查。
所有 Docker 的 iptables 规则都会被添加到 DOCKER 规则链中。不要手动操作这个规则链。如果你想在 Docker 的规则之前添加规则，可以添加到 DOCKER-USER
中。这些规则会在任何 Docker 自动创建的规则之前被使用。

在 Forward 规则链中的规则，不管是手动添加的或者其他基于 iptables 的防火墙应用添加的，会在上述的规则链后面才会被使用。
这就意味着通过 Docker 暴露的端口会无视防火墙的任何配置，因此，如果你希望阻止某个端口通过 Docker 被暴露出来，
必须在 DOCKER-USER 规则链中添加对应的规则。

##### 限制对 Docker 机器的连接

默认情况下，任何外部主机都允许直接联入 Docker 容器，为了仅允许特定的 IP 或者网段访问容器，可以插入一个 DROP 规则在 DOCKER-USER 链中，如下，限制
外部所有主机的访问，`192.168.1.1` 除外：

```SHELL
$ iptables -I DOCKER-USER -i ext_if ! -s 192.168.1.1 -j DROP

````
你需要修改 ext_if 的值为对应外部端口的 IP 地址，同样，也可以设置一个网段：

```SHELL
$ iptables -I DOCKER-USER -i ext_if ! -s 192.168.1.0/24 -j DROP

```

最后，你也可以指定一个接受的 IP 地址的范围通过使用 `--src-range` (--src-range 和 --dst-range 需要配合 -m iprange 一起使用)

```SHELL
$ iptables -I DOCKER-USER -m iprange -i ext_if ! --src-range 192.168.1.1-192.168.1.3 -j DROP

```

你可以组合 -s 或者 --src-range 和 -d 或者 --dst-range 来同时控制源 IP 和 目的 IP，例如，你的 Docker daemon 同时监听了 `192.168.1.99`和
`10.1.2.3`，你可以对 `10.1.2.3` 指定规则，而 `192.168.1.99` 保持开放。

`iptables` 更加复杂的应用不是本文的重点。可以通过这里了解更多：[ Netfilter.org HOWTO](https://www.netfilter.org/documentation/HOWTO/NAT-HOWTO.html)


#### Docker On a router

Docker也会在 FORWARD 规则链中设置 DROP 策略。如果 Docker 所在的主机同时也作为一个路由主机，那么这会导致这台主机不会转发任何流量。
可以通过在 DOCKER-USER 链中显示指定 ACCEPT 规则来允许转发：

```SHELL
$ iptables -I DOCKER-USER -i src_if -o dst_if -j ACCEPT
```

#### 阻止 Docker 操作 iptables

It is possible to set the `iptables` key to `false` in the Docker engine’s configuration file at `/etc/docker/daemon.json`,
but this option is not appropriate for most users. 
It is not possible to completely prevent Docker from creating iptables rules, and creating them after-the-fact is extremely
involved and beyond the scope of these instructions. Setting iptables to false will more than likely break container
networking for the Docker engine.


#### 设置容器默认绑定地址

By default, the Docker daemon will expose ports on the 0.0.0.0 address, i.e. any address on the host. If you want to change
that behavior to only expose ports on an internal IP address, you can use the --ip option to specify a different IP address.
However, setting --ip only changes the `default`, it does not restrict services to that IP.













