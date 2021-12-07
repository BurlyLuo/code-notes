# Docker部署Grafana

## 介绍

开源监控可视化套件

## 部署

```bash
docker run  -d --name grafana -p 3000:3000 -v /path/grafana/data:/var/lib/grafana  grafana/grafana
```

grafana的配置文件为 /etc/grafana/grafana.ini ，可以进入容器进行修改，或者挂出到宿主机。

grafana官方模板：https://grafana.com/grafana/dashboards
