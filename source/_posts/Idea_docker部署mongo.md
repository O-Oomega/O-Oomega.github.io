---
title: docker部署mongo
date: 2024-03-28 22:00:00
categories:
- Idea
tags:
- 突发奇想

---

### 在宿主机上创建需要挂载的文件


```bash
# 创建文件在 /docker目录下 数据文件夹和日志文件夹
mkdir -p /docker/mongodb/{data,logs}
```

### 运行容器


```bash
docker run -d --name mongodb -p 27017:27017 -v /docker/mongodb/data:/data/db -v /docker/mongodb/log:/data/log -e MONGO_INITDB_ROOT_USERNAME=root -e  MONGO_INITDB_ROOT_PASSWORD=root_pw mongo
```
以下是对每个参数的解释：

- `-d`: 这是 Docker 命令的一个选项，表示在后台（detached）模式下运行容器。

- `--name mongodb`: 这是指定容器的名称为 `mongodb`。

- `-p 27017:27017`: 这是端口映射，将主机的 27017 端口映射到容器的 27017 端口，用于 MongoDB 的客户端连接。

- `-v /docker/mongodb/data:/data/db`: 这是将主机上的 `/docker/mongodb/data` 目录挂载到容器内的 `/data/db` 目录，用于存储 MongoDB 数据。

- `-v /docker/mongodb/log:/data/log`: 这是将主机上的 `/docker/mongodb/log` 目录挂载到容器内的 `/data/log` 目录，用于存储 MongoDB 日志。

- `-e MONGO_INITDB_ROOT_USERNAME=root`: 这是设置 MongoDB 初始化数据库时的根用户名为 `root`。

- `-e MONGO_INITDB_ROOT_PASSWORD=root_pw`: 这是设置 MongoDB 初始化数据库时的根用户密码为 `root_pw`。

- `mongo`: 这是指定要使用的 MongoDB 镜像。

以下是将该 Docker 命令转换为 Docker Compose 的 YAML 文件：

```yaml
version: '3.8'

services:
  mongodb:
    image: mongo
    container_name: mongodb
    ports:
      - "27017:27017"
    volumes:
      - /docker/mongodb/data:/data/db
      - /docker/mongodb/log:/data/log
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: root_pw
```

你可以将这个 YAML 内容保存到一个文件（比如 `docker-compose.yml`），然后在包含该文件的目录中运行 `docker-compose up -d` 命令来启动 MongoDB 服务。
