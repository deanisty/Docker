#### 问题

宿主机以非 root 用户运行容器后，默认容器内的用户都是 root (uid=0, gid=0)
如果在宿主机和容器间映射共享文件系统，并在容器内新增文件之后，文件所属用户均为root，宿主机此时便无法再对文件进行操作，因为没有权限。

#### Linux User Namespace

> Linux 用户名称空间 http://man7.org/linux/man-pages/man7/user_namespaces.7.html

docker 在 Docker Engine 1.10(2016-02-04)引入了 Linux 用户名称空间的功能 ，称之为 userns-remap

使用方法如下：



#### 参考

* [开启docker 中的User Namespace 来解决权限问题](https://www.binss.me/blog/solve-docker-permission-problem-by-using-user-namespace/)
* [Isolate containers with a user namespace](https://docs.docker.com/engine/security/userns-remap/)


#### 问题

##### 设置命名空间映射之后导致容器运行失败

[解决方案](https://success.docker.com/article/user-namespace-runtime-error)
