# 集群

Docker Swarm 是一个原生的Docker集群管理工具，Swarm将一组Docker主机作为一个虚拟的Docker主机来管理。Swarm有一个非常简单的架构，它将多台主机作为一个集群
并在集群级别上以标准的Docker API的形式提供服务。这非常强大，它将Docker容器抽象到集群级别，而又不需要重新学习一套新的API。这也使得Swarm非常容易和那些
已经集成了Docker的工具再次集成，包括标准的Docker客户端。对Docker客户端来说，Swarm集群不过是另一台普通的Docker主机而已。

### 安装Swarm

### 创建Swarm集群

集群发现服务  consul etcd zookeeper
