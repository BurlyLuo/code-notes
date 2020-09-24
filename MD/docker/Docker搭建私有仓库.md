# Docker私有仓库

## 私有仓库搭建与配置

1.拉取私有仓库镜像

```bash
docker pull registry
```

2.启动私有仓库容器

```bash
docker run ‐di ‐‐name=registry ‐p 5000:5000 registry
```

3.查询私有仓库

打开浏览器 输入地址<http://YOUR_IP:5000/v2/_catalog> 看到 {"repositories":[]} 表示私有仓库搭建成功并且内容为空

4.修改daemon.json

```bash
vi /etc/docker/daemon.json
```

添加以下内容，保存退出。

```bash
{"insecure‐registries":["YOUR_IP:5000"]}
```

此步用于让 docker信任私有仓库地址。

5.重启docker服务

```bash
systemctl restart docker
```

## 镜像上传至私有仓库

0.创建一个jdk8镜像

创建文件Dockerfile

```dockerfile
#依赖镜像名称和ID
FROM centos:7
#指定镜像创建者信息
MAINTAINER AUTHOR
#切换工作目录
WORKDIR /usr
RUN mkdir /usr/local/java
#ADD 是相对路径jar,把java添加到容器中
ADD jdk‐8u171‐linux‐x64.tar.gz /usr/local/java/
#配置java环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_171
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH $JAVA_HOME/bin:$PATH
```

执行命令构建镜像

```bash
docker build ‐t='jdk1.8' .
```

注意后边的空格和点，不要省略

查看镜像是否建立完成

```bash
docker images
```

1.标记此镜像为私有仓库的镜像

```bash
docker tag jdk1.8 YOUR_IP:5000/jdk1.8
```

2.再次启动私服容器

```bash
docker start registry
```

3.上传标记的镜像

```bash
docker push YOUR_IP:5000/jdk1.8
```
