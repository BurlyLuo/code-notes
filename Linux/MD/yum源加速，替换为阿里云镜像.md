# yum源加速，替换为阿里云镜像

首先备份一下原先的yum源，避免出错无法恢复

```bash
cd /etc/yum.repos.d/
mv CentOS-Base.repo CentOS-Base.repo.bak
```

修改base.reop源

```bash
wget -O CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

安装epel.repo源

```bash
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```

刷新缓存

```bash
yum clean all
yum makecache
```
