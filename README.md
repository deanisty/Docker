# Docker
关于 docker 的概念、用法速记

#### 简介

Docker通过虚拟容器技术，使应用程序更加安全的隔离运行，并且更加轻量。

#### 容器和虚拟机

当一个容器在 Linux 系统中运行，它和其他容器共享操作系统。它运行在一个独立的进程中，不会占用比其他可执行文件更多的内存，所以它更轻量级。
相比之下，一个虚拟机运行在一个完整的操作系统中（可以被称为访客操作系统），并且通过超级监管器(hypervisor layer)访问虚拟系统资源。 
通常情况下，虚拟机提供的资源比一个应用程序运行时需要的多。

#### 核心组件

* [Docker客户端和服务器（Docker引擎）](Client-Server.md)
* [Docker镜像（Image）](image.md)
* [Docker仓库（Registry）](registry.md)
* [Docker容器 （container）](container.md)


#### 实践

* [Docker的安装](http://www.dockerinfo.net/docker%E5%AE%89%E8%A3%85-centos)
* [Docker镜像操作](http://www.dockerinfo.net/image%E9%95%9C%E5%83%8F)
* [Docker容器操作](http://www.dockerinfo.net/docker%E5%AE%B9%E5%99%A8-2)
* [Docker网络操作](http://www.dockerinfo.net/%E4%BD%BF%E7%94%A8%E7%BD%91%E7%BB%9C)
* [Docker容器编排](http://www.dockerinfo.net/docker-compose-%E9%A1%B9%E7%9B%AE)
* [Docker集群](swarm.md)
* Remote API


#### Read More

[底层实现](http://www.dockerinfo.net/%E5%BA%95%E5%B1%82%E5%AE%9E%E7%8E%B0)
