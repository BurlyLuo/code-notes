# Docker部署ZooKeeper

ZooKeeper官网：[zookeeper.apache.org](https://zookeeper.apache.org/)

官方Docker hub中的ZooKeeper镜像地址：[ZooKeeper](https://hub.docker.com/_/zookeeper)

## 拉取镜像

```bash
docker pull zookeeper:3.6.1
```

## 单机模式

### 创建挂载目录

```bash
mkdir -p /opt/zookeeper/standalone/data
mkdir -p /opt/zookeeper/standalone/datalog
mkdir -p /opt/zookeeper/standalone/logs
```

### 启动容器

命令行输入启动命令：

```bash
docker run -d -p 2181:2181 \
    --name zookeeper_3.6.1 \
    --restart always \
    --privileged \
    -v /etc/localtime:/etc/localtime \
    --volume /opt/zookeeper/standalone/data:/data \
    --volume /opt/zookeeper/standalone/datalog:/datalog \
    --volume /opt/zookeeper/standalone/logs:/logs \
    zookeeper:3.6.1
```

执行命令后，可以使用下面的命令查看容器运行状态：

```bash
docker ps -a
```

看到容器的STATUS为Up时，部署成功。

### 查看ZooKeeper运行情况

进入zookeeper容器

```bash
docker exec -it zookeeper_3.6.1 bash
```

使用服务端工具检查服务器状态，在容器内输入：

```bash
root@b8ce73b1a94a:/apache-zookeeper-3.6.1-bin# ./bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: standalone
```

从验证结果中可以看到，为单机模式。

## 集群模式

伪集群，启动三个节点。

### 创建三个节点的挂载目录

```bash
mkdir -p /opt/zookeeper/cluster/node1/data
mkdir -p /opt/zookeeper/cluster/node1/datalog
mkdir -p /opt/zookeeper/cluster/node1/logs
mkdir -p /opt/zookeeper/cluster/node2/data
mkdir -p /opt/zookeeper/cluster/node2/datalog
mkdir -p /opt/zookeeper/cluster/node2/logs
mkdir -p /opt/zookeeper/cluster/node3/data
mkdir -p /opt/zookeeper/cluster/node3/datalog
mkdir -p /opt/zookeeper/cluster/node3/logs
```

### Docker中创建zookeeper网络

由于是伪集群，部署在同一台机器上，所以需要创建一个自定义网络，用于指定zookeeper的IP地址：

```bash
docker network create --subnet=172.18.0.0/16 subnet-zookeeper
```

## 启动三个节点容器

启动节点1：

```bash
docker run -d -p 2181:2181 \
    --name zookeeper_3.6.1_cluster_node1 \
    --net subnet-zookeeper --ip 172.18.0.11 \
    --restart always \
    -v /etc/localtime:/etc/localtime \
    --volume /opt/zookeeper/cluster/node1/data:/data \
    --volume /opt/zookeeper/cluster/node1/datalog:/datalog \
    --volume /opt/zookeeper/cluster/node1/logs:/logs \
    -e ZOO_MY_ID=1 \
    -e "ZOO_SERVERS=server.1=172.18.0.11:2888:3888;2181 server.2=172.18.0.12:2888:3888;2181 server.3=172.18.0.13:2888:3888;2181" \
    zookeeper:3.6.1
```

启动节点2：

```bash
docker run -d -p 2182:2181 \
    --name zookeeper_3.6.1_cluster_node2 \
    --net subnet-zookeeper --ip 172.18.0.12 \
    --restart always \
    -v /etc/localtime:/etc/localtime \
    --volume /opt/zookeeper/cluster/node2/data:/data \
    --volume /opt/zookeeper/cluster/node2/datalog:/datalog \
    --volume /opt/zookeeper/cluster/node2/logs:/logs \
    -e ZOO_MY_ID=2 \
    -e "ZOO_SERVERS=server.1=172.18.0.11:2888:3888;2181 server.2=172.18.0.12:2888:3888;2181 server.3=172.18.0.13:2888:3888;2181" \
    zookeeper:3.6.1
```

启动节点3：

```bash
docker run -d -p 2183:2181 \
    --name zookeeper_3.6.1_cluster_node3 \
    --net subnet-zookeeper --ip 172.18.0.13 \
    --restart always \
    -v /etc/localtime:/etc/localtime \
    --volume /opt/zookeeper/cluster/node3/data:/data \
    --volume /opt/zookeeper/cluster/node3/datalog:/datalog \
    --volume /opt/zookeeper/cluster/node3/logs:/logs \
    -e ZOO_MY_ID=3 \
    -e "ZOO_SERVERS=server.1=172.18.0.11:2888:3888;2181 server.2=172.18.0.12:2888:3888;2181 server.3=172.18.0.13:2888:3888;2181" \
    zookeeper:3.6.1
```

命令参数说明：

- ZOO_MY_ID：表示 ZK 服务的 id, 它是1-255 之间的整数, 必须在集群中唯一
- ZOO_SERVERS：ZK 集群的主机列表

### 验证集群部署

以交互方式进入三个节点容器中查看：

```bash
docker exec -it zookeeper_3.6.1_cluster_node1 /bin/bash
```

```bash
docker exec -it zookeeper_3.6.1_cluster_node2 /bin/bash
```

```bash
docker exec -it zookeeper_3.6.1_cluster_node3 /bin/bash
```

使用服务端工具检查服务器状态，在容器内输入：

```bash
# node1
root@61c02df1cf38:/apache-zookeeper-3.6.1-bin# ./bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower
```

```bash
# node2
root@5b9b20ae160e:/apache-zookeeper-3.6.1-bin# ./bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: leader
```

```bash
# node3
root@6e4b7529c768:/apache-zookeeper-3.6.1-bin# ./bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: follower
```

从验证结果可以看出：node1为跟随者，node2为领导者，node3为跟随者。集群部署成功。

## 其他

### Zookeeper 常用命令

进入容器后，可使用一些可执行脚本，在`/bin`目录下。

**zkServer**：

```bash
# 启动服务
./zkServer.sh start
```

```bash
# 停止服务
./zkServer.sh stop
```

```bash
# 重启服务
./zkServer.sh restart
```

```bash
# 执行状态
./zkServer.sh status
```

**zkClient**：

```bash
#客户端连接服务器并进入 Bash 模式
./zkCli.sh -server <ip>:<port>

# 命令参数
ZooKeeper -server host:port cmd args
    stat path [watch]
    set path data [version]
    ls path [watch]
    delquota [-n|-b] path
    ls2 path [watch]
    setAcl path acl
    setquota -n|-b val path
    history
    redo cmdno
    printwatches on|off
    delete path [version]
    sync path
    listquota path
    rmr path
    get path [watch]
    create [-s] [-e] path data acl
    addauth scheme auth
    quit
    getAcl path
    close
    connect host:port
```

```bash
# 创建节点（Bash 模式）
create /test "hello zookeeper"
```

```bash
# 查询节点（Bash 模式）
get /test
```

```bash
# 删除节点（Bash 模式）
delete /test
```

## 参考

- [Docker下安装zookeeper（单机 & 集群）](https://www.cnblogs.com/LUA123/p/11428113.html)
- [基于 Docker 安装 Zookeeper](https://www.jianshu.com/p/c486133a70e4)
