## 使用 docker-composer 期间遇到的问题及解决方案


#### docker-composer up -d --build 不重新编译的问题

这个问题遇到过两次，后来发现是 
从 git 仓库中拉出来的 docker-composer 项目中的 yml 文件中

##### 定义 service 的是 image 指令 不是 build ，所以不论怎么改

对应目录下的 Dockerfile 都不会触发重新编译，太坑了！！！！

#### docker 容器中使用 composer install/update 直接被 killed

通过 docker stats 容器名 实时查看容器占用资源情况

如果内存使用量并未超过 最大内容容量  则应该是 php 限制了程序使用内存太小

修改 php.ini memory_limit = xxM 调整内容到一个合适的值即可

另外 composer install/update -vvv 命令可以看到具体执行的日志 便于调试

#### docker 容器中使用 composer install 报 proc_open Out out memory 异常

执行  

```Shell
composer install --prefer-dist -vvv ???? 
```

这样可以成功，但是为什么 ------

>--prefer-dist would try to download and unzip archives of the dependencies using GitHub or another API when available. This is used for >faster downloading of dependencies in most cases. It doesn't download the whole VCS history of the dependencies and it should be better >cached. Also archives on GitHub could exclude some files you don't need for just using the dependency with .gitattributes exclude >directive.

>--prefer-source would try to clone and keep the whole VCS repository of the dependencies when available. This is useful when you want >to have the original VCS repositories cloned in your vendor/ folder. E.g. you might want to work on the dependencies - modify them, >fork them, submit pull requests etc. while also using them as part of the bigger project which requires them in the first place.


#### docker 容器 npm dev 服务无法访问 connection refused

在 docker 容器中使用  npm run dev 启动 dev 服务器成功，并且容器 8080端口也映射到 主机 8080 端口，但是服务无法访问

```SHELL
$ docker container ls
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                                      NAMES
dfbdab04e01b        node               "docker-entrypoint.s…"   13 minutes ago      Up 12 minutes       0.0.0.0:8080->8080/tcp                     node_1

```

看下[这篇文章](https://pythonspeed.com/articles/docker-connection-refused/)

大概意思就是：
1. npm dev 服务器监听在 127.0.0.1:8080 端口
2. 端口映射  0.0.0.0:8080->8080/tcp 指令会把容器外部机器的所有端口流量都重定向到容器的外部网络接口上，也就是 eth0 端口，但是跟容器的127.0.0.1端口
   没关系，所以无法访问到
3. 因此要调整 npm dev 服务器监听的地址为：0.0.0.0:8080 这样dev服务器会同时监听容器的对外网络接口，就能访问到了
