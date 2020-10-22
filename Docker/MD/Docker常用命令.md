# Docker常用命令

官方命令文档：[Docker官网文档-Docker CLI](https://docs.docker.com/engine/reference/run/)

官方命令文档：[Docker官网文档-Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

## Docker核心概念

最核心的是理解三个概念，分别是：

- 仓库（Registry）
- 镜像（image）
- 容器（Container）

**镜像**：

就和我上面说的使用光盘拷贝已经有的镜像一样，我们的镜像是指一个系统的镜像。

我们的镜像都是基于 linux 的，准确来说是基于 ubuntu 的。

docker 镜像可以理解为，你在 win 下用 ghost 拷贝出来的磁盘镜像。不过它是 linux 版的。

**容器**：

容器本身就是我们最重要的概念，我们使用 docker 要做的就是容器这个东西。

简单来说容器是一个镜像的实例，更通俗来说容器就像你用 vm 或者 virtualbox 使用镜像创建的一个虚拟机实例。

**仓库**：

仓库就是镜像仓库。

如果你写代码，你肯定就知道 github，我们把代码托管到 github 之上。

如果我们部署，肯定就要用 dockerhub，我们把镜像托管到 docker hub 上(当然我们也可以假设，或者是用别人假设的hub)

国内有很多三方 dockerhub 服务器， 有阿里云，网易蜂巢，有容云，daocloud 等等等等。

至于国外那就更多了，如果非要推荐一家，那就是 amazon 了，毕竟云服务他们家宇宙最强，没有之一，没有对手。

## Docker命令分类

- Docker环境信息 — docker [info|version]
- 容器生命周期管理 — docker [create|exec|run|start|stop|restart|kill|rm|pause|unpause]
- 容器操作运维 — docker [ps|inspect|top|attach|wait|export|port|rename|stat]
- 容器rootfs命令 — docker [commit|cp|diff]
- 镜像仓库 — docker [login|pull|push|search]
- 本地镜像管理 — docker [build|images|rmi|tag|save|import|load]
- 容器资源管理 — docker [volume|network]
- 系统日志信息 — docker [events|history|logs]

## Docker命令结构图

![Docker命令结构图](/Docker/IMG/001.png)

## Docker环境信息

**info命令**：

用于检测Docker是否正确安装，一般结合docker version命令使用。

**version命令**：

查看 Docker 版本信息。

## 容器生命周期管理

**从image启动一个container（run）**：

docker run命令首先会从特定的image创之上create一层可写的container，然后通过start命令来启动它。停止的container可以重新启动并保留原来的修改。run命令启动参数有很多，以下是一些常规使用说明。

当利用 docker run 来创建容器时，Docker 在后台运行的标准操作包括：

- 检查本地是否存在指定的镜像，不存在就从公有仓库下载
- 利用镜像创建并启动一个容器
- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
- 从地址池配置一个 ip 地址给容器
- 执行用户指定的应用程序
- 执行完毕后容器被终止
- Usage: docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

> 使用image创建container并执行相应命令，然后停止

```bash
$ docker run ubuntu echo "hello world"
hello word
```

这是最简单的方式，跟在本地直接执行echo 'hello world' 几乎感觉不出任何区别，而实际上它会从本地ubuntu:latest镜像启动到一个容器，并执行打印命令后退出（docker ps -l可查看）。需要注意的是，默认有一个--rm=true参数，即完成操作后停止容器并从文件系统移除。因为Docker的容器实在太轻量级了，很多时候用户都是随时删除和新创建容器。
容器启动后会自动随机生成一个CONTAINER ID，这个ID在后面commit命令后可以变为IMAGE ID。

> 映射host到container的端口和目录：

映射主机到容器的端口是很有用的，比如在container中运行memcached，端口为11211，运行容器的host可以连接container的 internel_ip:11211 访问，如果有从其他主机访问memcached需求那就可以通过-p选项，形如`-p <host_port:contain_port>`，存在以下几种写法：

- -p 11211:11211 这个即是默认情况下，绑定主机所有网卡（0.0.0.0）的11211端口到容器的11211端口上
- -p 127.0.0.1:11211:11211 只绑定localhost这个接口的11211端口
- -p 127.0.0.1::5000
- -p 127.0.0.1:80:8080

目录映射其实是“绑定挂载”host的路径到container的目录，这对于内外传送文件比较方便，为了避免私服container停止以后保存的images不被删除，就要把提交的images保存到挂载的主机目录下。
使用比较简单，`-v <host_path:container_path>`，绑定多个目录时再加-v。

```bash
-v /tmp/docker:/tmp/docker
```

如果你共享的是多级的目录，可能会出现权限不足的提示。 这是因为CentOS7中的安全模块selinux把权限禁掉了，我们需要添加参数 -privileged=true 来解决挂载的目录没有权限的问题。

> 参数说明

-i：表示运行容器

-t：表示容器启动后会进入其命令行。加入这两个参数后，容器创建就能登录进去。即分 配一个伪终端。

--name :为创建的容器命名。

-v：表示目录映射关系（前者是宿主机目录，后者是映射到宿主机上的目录），可以使 用多个－v做多个目录或文件映射。注意：最好做目录映射，在宿主机上做修改，然后共 享到容器上。

-d：在run后面加上-d参数,则会创建一个守护式容器在后台运行（这样创建容器后不会 自动登录容器，如果只加-i -t两个参数，创建后就会自动进去容器）。

-p：表示端口映射，前者是宿主机端口，后者是容器内的映射端口。可以使用多个-p做 多个端口映射

> 交互式方式创建容器

```bash
docker run ‐it ‐‐name=容器名称 镜像名称:标签 /bin/bash
```

退出当前容器:

```bash
exit
```

> 守护式方式创建容器

```bash
docker run ‐di ‐‐name=容器名称 镜像名称:标签
```

登录守护式容器方式：

```bash
docker exec ‐it 容器名称 (或者容器ID) /bin/bash
```

**开启/停止/重启container（start/stop/restart）**：

容器可以通过run新建一个来运行，也可以重新start已经停止的container，但start不能够再指定容器启动时运行的指令，因为docker只能有一个前台进程。
容器stop（或Ctrl+D）时，会在保存当前容器的状态之后退出，下次start时保有上次关闭时更改。而且每次进入attach进去的界面是一样的，与第一次run启动或commit提交的时刻相同。

```bash
docker stop $CONTAINER_ID
docker restart $CONTAINER_ID
```

**删除一个或多个container、image（rm、rmi）**：

你可能在使用过程中会build或commit许多镜像，无用的镜像需要删除。但删除这些镜像是有一些条件的：

- 同一个IMAGE ID可能会有多个TAG（可能还在不同的仓库），首先你要根据这些 image names 来删除标签，当删除最后一个tag的时候就会自动删除镜像；
- 承上，如果要删除的多个IMAGE NAME在同一个REPOSITORY，可以通过`docker rmi <image_id>`来同时删除剩下的TAG；若在不同Repo则还是需要手动逐个删除TAG；
- 还存在由这个镜像启动的container时（即便已经停止），也无法删除镜像；

## 容器运维操作

**查看容器的信息container（ps）**：

`docker ps`命令可以查看容器的CONTAINER ID、NAME、IMAGE NAME、端口开启及绑定、容器启动后执行的COMMNAD。最常用的功能是通过ps来找到CONTAINER_ID，以便对特定容器进行操作。
`docker ps`默认显示当前正在运行中的container
`docker ps -a`查看包括已经停止的所有容器
`docker ps -l`显示最新启动的一个容器（包括已停止的）

**inspect命令**：

用于查看镜像和容器的详细信息，默认会列出全部信息，可以通过--format参数来指定输出的模板格式，以便输出特定信息。

inspect的对象可以是image、运行中的container和停止的container。

查看容器的内部IP：

```bash
docker inspect --format='{{.NetworkSettings.IPAddress}}' $CONTAINER_ID
```

**attach命令**：

docker attach命令对应开发者很有用，可以连接到正在运行的容器，观察容器的运行状况，或与容器的主进程进行交互。

要attach上去的容器必须正在运行，可以同时连接上同一个container来共享屏幕（与screen命令的attach类似）。
官方文档中说attach后可以通过CTRL-C来detach，但实际上经过我的测试，如果container当前在运行bash，CTRL-C自然是当前行的输入，没有退出；如果container当前正在前台运行进程，如输出nginx的access.log日志，CTRL-C不仅会导致退出容器，而且还stop了。这不是我们想要的，detach的意思按理应该是脱离容器终端，但容器依然运行。好在attach是可以带上--sig-proxy=false来确保CTRL-D或CTRL-C不会关闭容器。

```bash
docker attach --sig-proxy=false $CONTAINER_ID
```

**查看容器中正在运行的进程（top）**：

容器运行时不一定有`/bin/bash`终端来交互执行top命令，查看container中正在运行的进程，况且还不一定有top命令，这是`docker top <container_id/container_name>`就很有用了。实际上在host上使用`ps -ef | grep docker`也可以看到一组类似的进程信息，把container里的进程看成是host上启动docker的子进程就对了。

## 容器rootfs命令

**将一个container固化为一个新的image（commit）**：

当我们在制作自己的镜像的时候，会在container中安装一些工具、修改配置，如果不做commit保存起来，那么container停止以后再启动，这些更改就消失了。

Usage: `docker commit <container> [repo:tag]`

后面的repo:tag可选

只能提交正在运行的container，即通过docker ps可以看见的容器。

**文件拷贝cp**：

如果我们需要将文件拷贝到容器内可以使用cp命令

```bash
docker cp 需要拷贝的文件或目录 容器名称:容器目录
```

也可以将文件从容器内拷贝出来

```bash
docker cp 容器名称:容器目录 需要拷贝的文件或目录
```

## 镜像仓库

**在docker index中搜索image（search）**：

Usage: docker search TERM

```bash
docker search nginx
NAME                              DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
nginx                             Official build of Nginx.                        12118               [OK]
jwilder/nginx-proxy               Automated Nginx reverse proxy for docker con…   1678                                    [OK]
richarvey/nginx-php-fpm           Container running Nginx + PHP-FPM capable of…   744                                     [OK]
linuxserver/nginx                 An Nginx container, brought to you by LinuxS…   79
bitnami/nginx                     Bitnami nginx Docker Image                      72                                      [OK]
tiangolo/nginx-rtmp               Docker image with Nginx using the nginx-rtmp…   58                                      [OK]
nginxdemos/hello                  NGINX webserver that serves a simple page co…   31                                      [OK]
jlesage/nginx-proxy-manager       Docker container for Nginx Proxy Manager        27                                      [OK]
jc21/nginx-proxy-manager          Docker container for managing Nginx proxy ho…   26
nginx/nginx-ingress               NGINX Ingress Controller for Kubernetes         22
privatebin/nginx-fpm-alpine       PrivateBin running on an Nginx, php-fpm & Al…   18                                      [OK]
schmunk42/nginx-redirect          A very simple container to redirect HTTP tra…   17                                      [OK]
blacklabelops/nginx               Dockerized Nginx Reverse Proxy Server.          12                                      [OK]
centos/nginx-18-centos7           Platform for running nginx 1.8 or building n…   12
centos/nginx-112-centos7          Platform for running nginx 1.12 or building …   10
nginxinc/nginx-unprivileged       Unprivileged NGINX Dockerfiles                  9
webdevops/nginx                   Nginx container                                 8                                       [OK]
nginx/nginx-prometheus-exporter   NGINX Prometheus Exporter                       7
sophos/nginx-vts-exporter         Simple server that scrapes Nginx vts stats a…   5                                       [OK]
1science/nginx                    Nginx Docker images that include Consul Temp…   5                                       [OK]
mailu/nginx                       Mailu nginx frontend                            4                                       [OK]
pebbletech/nginx-proxy            nginx-proxy sets up a container running ngin…   2                                       [OK]
ansibleplaybookbundle/nginx-apb   An APB to deploy NGINX                          1                                       [OK]
centos/nginx-110-centos7          Platform for running nginx 1.10 or building …   0
wodby/nginx                       Generic nginx                                   0                                       [OK]
```

搜索的范围是官方镜像和所有个人公共镜像。NAME列的 / 后面是仓库的名字。

NAME：仓库名称
DESCRIPTION：镜像描述
STARS：用户评价，反应一个镜像的受欢迎程度
OFFICIAL：是否官方
AUTOMATED：自动构建，表示该镜像由Docker Hub自动构建流程创建的

**从docker registry server 中下拉image或repository（pull）**：

Usage: `docker pull [OPTIONS] NAME[:TAG]`

```bash
docker pull centos
```

上面的命令需要注意，在docker v1.2版本以前，会下载官方镜像的centos仓库里的所有镜像，而从v.13开始官方文档里的说明变了：will pull the centos:latest image, its intermediate layers and any aliases of the same id，也就是只会下载tag为latest的镜像（以及同一images id的其他tag）。

也可以明确指定具体的镜像：

```bash
docker pull centos:centos6
```

当然也可以从某个人的公共仓库（包括自己是私人仓库）拉取，形如docker pull username/repository<:tag_name> ：

```bash
docker pull seanlook/centos:centos6
```

如果你没有网络，或者从其他私服获取镜像，形如`docker pull registry.domain.com:5000/repos:<tag_name>`

```bash
docker pull dl.dockerpool.com:5000/mongo:latest
```

**推送一个image或repository到registry（push）**：

与上面的pull对应，可以推送到Docker Hub的Public、Private以及私服，但不能推送到Top Level Repository。

```bash
docker push seanlook/mongo
docker push registry.tp-link.net:5000/mongo:2014-10-27
```

registry.tp-link.net也可以写成IP，如：172.29.88.222。
在repository不存在的情况下，命令行下push上去的会为我们创建为私有库，然而通过浏览器创建的默认为公共库。

## 本地镜像管理

**列出机器上的镜像（images）**：

```bash
$ docker images
REPOSITORY               TAG             IMAGE ID        CREATED         VIRTUAL SIZE
...
```

我们可以根据REPOSITORY来判断这个镜像是来自哪个服务器，如果没有 / 则表示官方镜像，类似于username/repos_name表示Github的个人公共库，类似于regsistory.example.com:5000/repos_name则表示的是私服。
IMAGE ID列其实是缩写，要显示完整则带上--no-trunc选项。

REPOSITORY：镜像名称
TAG：镜像标签
IMAGE ID：镜像ID
CREATED：镜像的创建日期（不是获取该镜像的日期）
SIZE：镜像大小 

这些镜像都是存储在Docker宿主机的/var/lib/docker目录下

**docker build 使用此配置生成新的image**：

build命令可以从Dockerfile和上下文来创建镜像：

Usage: `docker build [OPTIONS] PATH | URL | -`

上面的PATH或URL中的文件被称作上下文，build image的过程会先把这些文件传送到docker的服务端来进行的。
如果PATH直接就是一个单独的Dockerfile文件则可以不需要上下文；如果URL是一个Git仓库地址，那么创建image的过程中会自动git clone一份到本机的临时目录，它就成为了本次build的上下文。无论指定的PATH是什么，Dockerfile是至关重要的。

官方网站文档：[Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

**给镜像打上标签（tag）**：

tag的作用主要有两点：一是为镜像起一个容易理解的名字，二是可以通过docker tag来重新指定镜像的仓库，这样在push时自动提交到仓库。

将同一IMAGE_ID的所有tag，合并为一个新的：

```bash
docker tag <IMAGE ID> <REPOSITORY>
```

新建一个tag，保留旧的那条记录：

```bash
docker tag Registry/Repos:Tag New_Registry/New_Repos:New_Tag
```

**镜像备份(save)**：

我们可以通过以下命令将镜像保存为 tar 文件

```bash
docker save ‐o mynginx.tar nginx
```

**镜像恢复与迁移(load)**：

首先我们先删除掉nginx镜像 然后执行此命令进行恢复

```bash
docker load ‐i mynginx.tar
```

-i 输入的文件

执行后再次查看镜像，可以看到镜像已经恢复

## 容器资源管理

**数据卷(volume)**：

Docker的数据持久化主要有两种方式：

- bind mount
- volume

Docker的数据持久化即使数据不随着container的结束而结束，数据存在于host机器上——要么存在于host的某个指定目录中（使用bind mount），要么使用docker自己管理的volume（/var/lib/docker/volumes下）。

> bind mount

bind mount自docker早期便开始为人们使用了，用于将host机器的目录mount到container中。但是bind mount在不同的宿主机系统时不可移植的，比如Windows和Linux的目录结构是不一样的，bind mount所指向的host目录也不能一样。这也是为什么bind mount不能出现在Dockerfile中的原因，因为这样Dockerfile就不可移植了。

将host机器上当前目录下的host-data目录mount到container中的/container-data目录：

```bash
docker run -it -v $(pwd)/host-dava:/container-data alpine sh
```

有几点需要注意：

- host机器的目录路径必须为全路径(准确的说需要以/或~/开始的路径)，不然docker会将其当做volume而不是volume处理
- 如果host机器上的目录不存在，docker会自动创建该目录
- 如果container中的目录不存在，docker会自动创建该目录
- 如果container中的目录已经有内容，那么docker会使用host上的目录将其覆盖掉

> 使用volume

volume也是绕过container的文件系统，直接将数据写到host机器上，只是volume是被docker管理的，docker下所有的volume都在host机器上的指定目录下`/var/lib/docker/volumes`。

将my-volume挂载到container中的/mydata目录：

```bash
docker run -it -v my-volume:/mydata alpine sh
```

然后可以查看到给my-volume的volume：

```bash
docker volume inspect my-volume
[
    {
        "CreatedAt": "2018-03-28T14:52:49Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/my-volume/_data",
        "Name": "my-volume",
        "Options": {},
        "Scope": "local"
    }
]
```

可以看到，volume在host机器的目录为/var/lib/docker/volumes/my-volume/_data。此时，如果my-volume不存在，那么docker会自动创建my-volume，然后再挂载。

也可以不指定host上的volume：

```bash
docker run -it -v /mydata alpine sh
```

此时docker将自动创建一个匿名的volume，并将其挂载到container中的/mydata目录。匿名volume在host机器上的目录路径类似于：/var/lib/docker/volumes/300c2264cd0acfe862507eedf156eb61c197720f69e7e9a053c87c2182b2e7d8/_data

除了让docker帮我们自动创建volume，我们也可以自行创建：

```bash
docker volume create my-volume-2
```

然后将这个已有的my-volume-2挂载到container中:

```bash
docker run -it -v my-volume-2:/mydata alpine sh
```

需要注意的是，与bind mount不同的是，如果volume是空的而container中的目录有内容，那么docker会将container目录中的内容拷贝到volume中，但是如果volume中已经有内容，则会将container中的目录覆盖。

> Dockerfile中的VOLUME

在Dockerfile中，我们也可以使用VOLUME指令来申明contaienr中的某个目录需要映射到某个volume：

```bash
#Dockerfile
VOLUME /foo
```

这表示，在docker运行时，docker会创建一个匿名的volume，并将此volume绑定到container的/foo目录中，如果container的/foo目录下已经有内容，则会将内容拷贝的volume中。也即，Dockerfile中的VOLUME /foo与docker run -v /foo alpine的效果一样。

Dockerfile中的VOLUME使每次运行一个新的container时，都会为其自动创建一个匿名的volume，如果需要在不同container之间共享数据，那么我们依然需要通过docker run -it -v my-volume:/foo的方式将/foo中数据存放于指定的my-volume中。

因此，VOLUME /foo在某些时候会产生歧义，如果不了解的话将导致问题。

**容器互联(network)**：

官方文档：

docker network所有子命令如下：

- docker network create 创建网络
- docker network connect
- docker network ls
- docker network rm
- docker network disconnect
- docker network inspect

> 创建网络

在安装Docker Engine时会自动创建一个默认的bridge网络docker0。
此外，还可以创建自己的bridge网络或overlay网络。

bridge网络依附于运行Docker Engine的单台主机上，而overlay网络能够覆盖运行各自Docker Engine的多主机环境中。

创建bridge网络比较简单如下：[Configure networking](https://docs.docker.com/network/)

```bash
 # 不指定网络驱动时默认创建的bridge网络
 docker network create simple-network
 # 查看网络内部信息
 docker network inspect simple-network
 # 应用到容器时，可进入容器内部使用ifconfig查看容器的网络详情
```

但是创建一个overlay网络就需要一些前提条件：

- key-value store（Engine支持Consul、Etcd和ZooKeeper等分布式存储的key-value store）
- 集群中所有主机已经连接到key-value store
- swarm集群中每个主机都配置了下面的daemon参数
- –cluster-store
- –cluster-store-opt
- –cluster-advertise

然后创建overlay网络：

```bash
# 创建网络时，使用参数`-d`指定驱动类型为overlay
docker network create -d overlay my-multihost-network
```

就使用--subnet选项创建子网而言，bridge网络只能指定一个子网，而overlay网络支持多个子网。

在bridge和overlay网络驱动下创建的网络可以指定不同的参数，详见官方文档。

> 连接容器

创建三个容器，分别前两个使用默认网络启动容器，第三个使用自定义bridge网络启动。
然后再将第二个容器添加到自定义网络。这三个容器的网络情况如下

- 第一个容器：只有默认的docker0
- 第二个容器：属于两个网络——docker0、自定义网络
- 第三个容器：只属于自定义网络

说明：通过容器启动指定的网络会覆盖默认bridge网络docker0。

```bash
# 创建三个容器 conTainer1,container2,container3
docker run -itd --name=container1 busybox
docker run -itd --name=container2 busybox
# 创建网络mynet
docker network create -d bridge --subnet 172.25.0.0/16 mynet
# 将容器containerr2连接到新建网络mynet
docker network connect mynet container2
# 使用mynet网络来容器container3
docker run --net=mynet --ip=172.25.3.3 -itd --name=container3 busybox

# 查看这三个容器的网络情况
docker network inspect container1 # docker0
docker network inspect container2 # docker0, mynet
docker network inspect container3 # mynet
```

> 默认网络与自定义bridge网络的差异

默认网络docker0：网络中所有主机间只能用IP相互访问。通过--link选项创建的容器可以对链接的容器名(container-name)作为hostname进行直接访问。

自定义网络(bridge)：网络中所有主机除ip访问外，还可以直接用容器名(container-name)作为hostname相互访问。

```bash
# 进入container2内部
docker attach container2
ping -w 4 container3 # 可访问
ping -w 4 container1 # 不可访问
ping -w 4 172.17.0.2 # 可访问container1的IP
# Ctrl+P+Q退出容器，让container2以守护进程运行
```

> 默认网络与自定义bridge网络在容器连接的差别

在默认网络中使用link（legency link），有如下功能：

- 使用容器名作为hostname
- link容器时指定`alias: --link=<Container-Name>:<Alias>`
- 配合--icc=false隔离性，实现容器间的安全连接
- 环境变量注入

自定义网络中使用docker net提供如下功能：

- 使用DNS实现自动化的名称解析
- 一个网络提供容器的安全隔离环境
- 动态地attach与detach到多个网络
- 支持与--link选项一起使用，为链接的容器提供别名（可以是尚不存在链接容器，与默认容器中–link使用的最大差别）

默认网络中的link是静态的，不允许链接容器重启，而自定义网络下的link是动态的，支持链接容器重启（以及IP变化）

因此，使用--link时链接的容器，在默认网络中必须提前创建好，而自定义网络下不必预先建好。

使用docker network connetct将容器连接到新网络中时，用参数--link链接相同的容器时，可以指定不同的别名，它们是针对不同网络的。

```bash
# 运行容器使用自定义网络，同时使用--link链接尚不存在的container5容器
docker run --net=mynet -itd --name=container4 --link container5:c5 busybox
# 创建容器container5
docker run --net=mynet -itd --name=container5 --link container4:c4 busybox
# 虽然是相同容器，但是在不同的网络环境连接中可以不同的alias链接
docker network connect --link container5:foo local_alias container4
docker network connect --link container4:bar local_alias container5
```

> 指定容器在网络范围的别名（Network-scoped alias）

Network-scoped alias就是指定容器在可被同一网络范围内的其他容器访问的别名。
不同于link别名的是，link别名是由链接容器的使用者提供的，只有它自己可使用；而指定网络范围内别名，是由容器提供给网络中其它容器使用的。

Network-scoped alias：同一网络中的多个容器可以指定相同的别名，在使用的当然只有第一个指定别名的容器才生效，只有当第一个容器关闭时，指定相同别名的第二个容器的别名才会开始生效。

```bash
docker run --net=mynet -itd --name=container6 --net-alias app busybox
docker network connect --alias scoped-app local_alias container6
docker run --net=isolated_nw -itd --name=container7 --net-alias app busybox
docker network connect --alias scoped-app local_alias container7
# 在container4中
docker attach container4
ping app # 访问container6的IP
# 从container4中以守护进程运行退出：Ctrl+P+Q
docker stop container6
docker attach container4
ping app # 访问的container7的IP
```

> 断开网络与移除网络

```bash
# 容器从mynet网络中断开（它将无法再网络中的容器container3通讯）
docker network disconnect mynet container2
# 测试与容器container3失败
docker attach container2
ping contianer3 # 访问失败
```

在多主机的网络环境中，在将容器用已移除的容器名称连接到网络中时会出现container already connected to network的错误，这时需要将新容器强制移除docker rm -f，重新运行并连接到网络中。

移除网络要求网络中所有的容器关闭或断开与此网络的连接时，才能够使用移除命令：

```bash
# 断开最后一个连接到mynet网络的容器
docker network disconnet mynet container3
# 移除网络
docker network rm mynet
```

## 系统日志信息

**events、history和logs命令**：

这3个命令用于查看Docker的系统日志信息。events命令会打印出实时的系统事件；history命令会打印出指定镜像的历史版本信息，即构建该镜像的每一层镜像的命令记录；logs命令会打印出容器中进程的运行日志。

`docker events [options]` ：从服务器获取实时事件。

> Options说明:
>
>-f ：根据条件过滤事件；
>
>--since ：从指定的时间戳后显示所有事件;
>
>--until ：流水时间显示到指定的时间为止；

`docker history [options] image`：查看指定镜像的创建历史。

> Options说明:
>
>-H :以可读的格式打印镜像大小和日期，默认为true；
>
>--no-trunc :显示完整的提交记录；
>
>-q :仅列出提交记录ID。

`docker logs [options] container`

> Options说明:
> --details        显示更多的信息
> -f, --follow         跟踪日志输出，最后一行为当前时间戳的日志
> --since string   显示自具体某个时间或时间段的日志
> --tail string    从日志末尾显示多少行日志， 默认是all
> -t, --timestamps     显示时间戳
