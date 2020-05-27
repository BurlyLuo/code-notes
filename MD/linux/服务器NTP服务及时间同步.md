# 服务器NTP服务及时间同步

## 情况描述

有几台Linux服务器，但是不能访问外网，长时间的运行下，发现服务器上的时间不太准确了，这时候就需要进行时间同步了。

## NTP时间同步方式选择

NTP同步方式在linux下一般两种：`使用ntpdate命令直接同步`和`使用NTPD服务平滑同步`。

`使用ntpdate命令直接同步`的方式存在风险，比如一些定时任务在已有时间内执行过，直接同步导致时间变回任务执行前的时间段，定时任务会重复执行。

`使用NTPD服务平滑同步`不会让一个时间点在一天内经历两次，而是平滑同步时间，它每次同步时间的偏移量不会太陡，是慢慢来的。

## 安装及配置NTPD服务

一般CentOS系统自带了该服务，不过可以检查是否安装，通过

```bash
rpm -q ntp
```

如果有输出ntp版本信息，即已安装。

配置NTPD服务的服务器需要能访问外网，这里挑选了一台可以访问外网的Linux服务器配置内网的NTPD服务，作为NTP-Server，其他几台内网通过它来进行时间同步。

这里假设其IP为192.168.1.1，其他几台内网的服务器IP分别为192.168.1.2、192.168.1.3。

在配置NTPD服务之前，先手动同步一下时间：

```bash
ntpdate cn.pool.ntp.org
```

配置NTP服务为自启动：

```bash
systemctl enable ntpd.service
```

## 配置内网NTP-Server(192.168.1.1)

修改NTPD服务的配置文件`/etc/ntp.conf`：

```properties
driftfile  /var/lib/ntp/drift
pidfile    /var/run/ntpd.pid
logfile    /var/log/ntp.log


# Access Control Support
restrict    default kod nomodify notrap nopeer noquery
restrict -6 default kod nomodify notrap nopeer noquery
restrict 127.0.0.1
restrict 192.168.0.0 mask 255.255.0.0 nomodify notrap nopeer noquery
restrict 172.16.0.0 mask 255.240.0.0 nomodify notrap nopeer noquery
restrict 100.64.0.0 mask 255.192.0.0 nomodify notrap nopeer noquery
restrict 10.0.0.0 mask 255.0.0.0 nomodify notrap nopeer noquery


# 允许内网其他机器同步时间
restrict 192.168.1.2 mask 255.255.255.0 nomodify notrap
restrict 192.168.1.3 mask 255.255.255.0 nomodify notrap

# local clock
# 外部时间服务器不可用时，以本地时间作为时间服务
server 127.0.0.1
fudge  127.0.0.1 stratum 10

# 中国这边最活跃的时间服务器 : http://www.pool.ntp.org/zone/cn
server 210.72.145.44 perfer      # 中国国家受时中心
server 202.112.10.36             # 1.cn.pool.ntp.org
server 59.124.196.83             # 0.asia.pool.ntp.org

disable monitor
```

配置文件修改完成，保存退出，启动服务。

```bash
service ntpd start
```

启动后，一般需要5-10分钟左右的时候才能与外部时间服务器开始同步时间。可以通过命令查询NTPD服务情况。

```bash
$ netstat -tlunp | grep ntp
udp        0      0 192.168.1.1:123         0.0.0.0:*                           14962/ntpd
udp        0      0 127.0.0.1:123           0.0.0.0:*                           14962/ntpd
udp        0      0 0.0.0.0:123             0.0.0.0:*                           14962/ntpd
udp6       0      0 :::123                  :::*                                14962/ntpd  
```

启动后，可通过 `ntpstat` 命令查看时间同步状态。

```bash
$ ntpstat
synchronised to NTP server (100.64.8.9) at stratum 3
   time correct to within 28 ms
   polling server every 64 s
```

## 配置内网NTP-Clients

内网其他设备作为NTP的客户端配置，相对就比较简单，而且所有设备的配置都相同。

首先需要安装NTPD服务，然后配置为自启动（与NTP-Server完全一样）。然后找其中一台配置/etc/ntp.conf文件，配置完成验证通过后，拷贝到其他客户端机器，直接使用即可。

```properties
driftfile  /var/lib/ntp/drift
pidfile    /var/run/ntpd.pid
logfile    /var/log/ntp.log


# Access Control Support
restrict    default kod nomodify notrap nopeer noquery
restrict -6 default kod nomodify notrap nopeer noquery
restrict 127.0.0.1
restrict 192.168.0.0 mask 255.255.0.0 nomodify notrap nopeer noquery
restrict 172.16.0.0 mask 255.240.0.0 nomodify notrap nopeer noquery
restrict 100.64.0.0 mask 255.192.0.0 nomodify notrap nopeer noquery
restrict 10.0.0.0 mask 255.0.0.0 nomodify notrap nopeer noquery
restrict 192.168.1.1 nomodify notrap noquery


# 配置时间服务器为本地的时间服务器
server 192.168.1.1


# local clock
# 外部时间服务器不可用时，以本地时间作为时间服务
server 127.0.0.1
fudge  127.0.0.1 stratum 10


disable monitor
```

保存退出，请求服务器前，先使用ntpdate手动同步下时间：

```bash
ntpdate -u 192.168.1.1
```

这里有可能出现同步失败，一般情况下原因都是本地的NTPD服务器还没有正常启动起来，一般需要几分钟时间后才能开始同步。

手动同步成功后，启动服务：

```bash
service ntpd start
```
