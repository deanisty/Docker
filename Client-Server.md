# 客户端-服务器模式

Docker是一个客户端/服务器架构的程序。

服务器以守护进程的形式运行，称作 Docker 引擎。

Docker提供一个命令行工具 docker 以及一套完整的 RESTful API 来与守护进程交互。

用户可以在同一台宿主机上运行 Docker 守护进程和客户端，也可以使用本地的客户端连接到运行在另一台宿主机上的远程 Docker 守护进程。

客户端向服务器发送请求，服务器完成工作并返回结果。

![docker-engine](https://docs.docker.com/engine/images/engine-components-flow.png)
