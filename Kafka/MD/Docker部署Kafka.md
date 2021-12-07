# Docker部署kafka

## 资料

dockerhub第三方仓库地址: https://hub.docker.com/r/wurstmeister/kafka

## 部署

启动zookeeper

```bash
docker run -d --name zookeeper -p 2181:2181 -t wurstmeister/zookeeper
```

启动kafka

```bash
docker run -d --name kafka \
-p 9092:9092 \
-e KAFKA_BROKER_ID=0 \
-e KAFKA_ZOOKEEPER_CONNECT=<your-ip>:2181 \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://<your-ip>:9092 \
-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 wurstmeister/kafka
```

启动kafka-manager

```bash
docker run -d -p 9000:9000 -e ZK_HOSTS=<your-ip>:2181 kafkamanager/kafka-manager
```
