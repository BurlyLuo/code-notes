# Docker部署Prometheus

注：可使用Prometheus + Grafana 监控 JVM

prometheus.yml

```yaml
global:
  scrape_interval:     60s
  evaluation_interval: 60s
 
scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['127.0.0.1:9090']
        labels:
          instance: prometheus
  - job_name: 'pushgateway'
    static_configs:
      - targets: ['127.0.0.1:9091']
        labels:
          instance: pushgateway
```

启动 prometheus pushgateway

```bash
docker run -d --name pushgateway -p 9091:9091 bitnami/pushgateway:latest
```

启动 prometheus

```bash
docker run -d --name prometheus \
    -p 9090:9090 \
    -v /path/prometheus/prometheus.yml:/opt/bitnami/prometheus/conf/prometheus.yml \
    -v /path/prometheus/data:/opt/bitnami/prometheus/data \
    bitnami/prometheus:latest
```
