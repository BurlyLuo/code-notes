# Docker配置web UI

这里我选择的是portainer，一个非常好用的docker ui：

- [Github地址](https://github.com/portainer/portainer)
- [Docs地址](https://portainer.readthedocs.io/en/latest/)
- [官方镜像](https://hub.docker.com/r/portainer/portainer)

## 下载镜像

```bash
docker pull portainer/portainer:1.23.0
```

## 创建目录

```bash
cd ~
mkdir portainer
cd portainer
mkdir data
```

## 启动运行

```bash
cd ~
docker run -d -it --restart always\
 -p 9000:9000 \
 --name portainer \
 -v /var/run/docker.sock:/var/run/docker.sock \
 -v $PWD/portainer/data:/data \
 portainer/portainer:1.23.0
```

## 使用说明

1.设置admin密码:

![示例](/Docker/IMG/008.png)

2.选择本地模式，当然也可以根据响应情况选择：

![示例](/Docker/IMG/009.png)

这里看到为什么启动命令中包含`-v /var/run/docker.sock:/var/run/docker.sock`，原因就是本地模式需要指定这个参数。

安装配置完成，可以使用了：

![示例](/Docker/IMG/010.png)

## 参考

- [portainer 神好用的docker ui](https://en.llycloud.com/archives/891)
