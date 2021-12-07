# Docker部署WordPress

Docker官网镜像地址：[wordpress](https://hub.docker.com/_/wordpress)

## 创建目录

```bash
mkdir -p /opt/wordpress
```

## 创建docker-compose.yaml

```bash
cd /opt/wordpress
vi docker-compose.yaml
```

docker-compose.yaml内容如下：

```yaml
version: '3.3'

services:
  db:
    image: mysql:5.7
    volumes:
      - dbdata:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: wordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress

volumes:
    dbdata:
```

注意：这里`dbdata`并不是在当前目录下，这个是`卷标`，需要通过它去查看具体数据存放地址：

```bash
#查看所有卷标
docker volume ls

DRIVER              VOLUME NAME
local               ac872199678fc6a95085209b70f7a55cd56ce1ba2b0768f554c966f1dac01f7e
local               wordpress_dbdata
```

这里`wordpress_dbdata`就是完整的卷标，再查看卷对应的地址：

```bash
docker volume inspect wordpress_dbdata

[
    {
        "CreatedAt": "2020-01-22T15:32:27+08:00",
        "Driver": "local",
        "Labels": {
            "com.docker.compose.project": "wordpress",
            "com.docker.compose.version": "1.25.1",
            "com.docker.compose.volume": "dbdata"
        },
        "Mountpoint": "/var/lib/docker/volumes/wordpress_dbdata/_data",
        "Name": "wordpress_dbdata",
        "Options": null,
        "Scope": "local"
    }
]
```

这样，就可以看到数据卷对应的地址了。

### 运行

```bash
docker-compose up -d
```

显示如下信息，8000端口被占用，则运行成功：

```text
Creating wordpress_db_1 ... done
Creating wordpress_wordpress_1 ... done
```

网页打开 `http://127.0.0.1:8000` 即可访问WordPress。
