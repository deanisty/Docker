## 使用 docker-composer 期间遇到的问题及解决方案


#### docker-composer up -d --build 不重新编译的问题

这个问题遇到过两次，后来发现是 
从 git 仓库中拉出来的 docker-composer 项目中的 yml 文件中

##### 定义 service 的是 image 指令 不是 build ，所以不论怎么改

对应目录下的 Dockerfile 都不会触发重新编译，太坑了！！！！