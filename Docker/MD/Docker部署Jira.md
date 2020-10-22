# Docker部署Jira

官方镜像：[atlassian-jira-software](https://hub.docker.com/r/cptactionhank/atlassian-jira-software)

## 拉取镜像

```bash
docker pull cptactionhank/atlassian-jira-software:8.1.0
```

## 创建目录

```bash
cd ~
mkdir jira
cd jira
mkdir data
```

## 启动运行

```bash
cd ~
docker run -it --detach --name jira-8.1.0 \
 --publish 8080:8080 \
 --privileged \
 -v $PWD/jira/data:/var/atlassian/jira/data \
 -v /etc/localtime:/etc/localtime \
 cptactionhank/atlassian-jira-software:8.1.0
```

## 参考

- [Docker 部署 Jira8.1.0](https://www.cnblogs.com/tchua/p/10862670.html)
