# Docker部署RabbitMQ

官方镜像：[rabbitmq](https://hub.docker.com/_/rabbitmq)

## 拉取镜像

```bash
docker pull rabbitmq:3.8.1-management
```

这里选择带managerment

## 创建目录

```bash
cd ~
mkdir rabbitmq
cd rabbitmq
mkdir data
```

## 启动运行

```bash
cd ~
docker run -d --name rabbitmq-3.8.1-management -p 5672:5672 -p 15672:15672 -v $PWD/rabbitmq/data:/var/lib/rabbitmq -e RABBITMQ_DEFAULT_VHOST=my_vhost  -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin rabbitmq:3.8.1-management
```

命令说明：

- -d 后台运行容器；
- --name 指定容器名；
- -p 指定服务运行的端口（5672：应用访问端口；15672：控制台Web端口号）；
- -v 映射目录或文件；
- -e 指定环境变量；（RABBITMQ_DEFAULT_VHOST：默认虚拟机名、RABBITMQ_DEFAULT_USER：默认的用户名、RABBITMQ_DEFAULT_PASS：默认用户名的密码）

访问15672端口，即可打开rabbitmq管理页面。
