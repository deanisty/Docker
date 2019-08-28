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

这样可以成功，但是为什么
