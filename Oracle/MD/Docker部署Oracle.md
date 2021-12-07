# Docker部署Oracle

- [Docker部署Oracle](#docker部署oracle)
  - [拉取镜像](#拉取镜像)
  - [启动容器](#启动容器)
  - [进入镜像进行配置](#进入镜像进行配置)
    - [进行软连接](#进行软连接)
    - [编辑profile文件配置ORACLE环境变量](#编辑profile文件配置oracle环境变量)
    - [创建软连接](#创建软连接)
    - [登录sqlplus并修改sys、system用户密码](#登录sqlplus并修改syssystem用户密码)
  - [远程链接](#远程链接)

## 拉取镜像

```bash
docker pull registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
```

## 启动容器

```bash
docker run -it -d -p 1521:1521 \
--name oracle11 \
-v /path/to/docker/oracle/data:/data/oracle \
registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
```

## 进入镜像进行配置

docker exec -it oracle11 bash

### 进行软连接

```bash
sqlplus /nolog
```

发现没有该命令，所以切换root用户。

```bash
su root
```

输入密码：helowin

### 编辑profile文件配置ORACLE环境变量

vi /etc/profile ，在文件最后写上下面内容

```bash
export ORACLE_HOME=/home/oracle/app/oracle/product/11.2.0/dbhome_2
export ORACLE_SID=helowin
export PATH=$ORACLE_HOME/bin:$PATH
```

保存后执行 `source /etc/profile` 加载环境变量

### 创建软连接

```bash
ln -s $ORACLE_HOME/bin/sqlplus /usr/bin
```

切换到 oracle 用户

```bash
su oracle
```

### 登录sqlplus并修改sys、system用户密码

```bash
sqlplus /nolog
conn /as sysdba

alter user system identified by system;
alter user sys identified by system;
create user test identified by test;
grant connect,resource,dba to test;
ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME UNLIMITED; 
alter system set processes=1000 scope=spfile;
```

修改以上信息后，需要重新启动数据库；

conn /as sysdba
-- 关闭数据库
shutdown immediate;
-- 启动数据库
startup;
-- 退出软链接
exit

## 远程链接

jdbc:oracle:thin:@//127.0.0.1:1521/helowin
test/test
