在 Windows 上编写 Dockerfile 来编译最新版本的 Redis 源码,并在远程 Docker 服务器上构建镜像,你可以按照以下步骤进行:

1. 在 Windows 上安装 Git 和 Docker,并确保可以连接到远程 Docker 服务器。

2. 从 Redis 的 GitHub 仓库克隆最新版本的源码:
   ```
   git clone https://github.com/redis/redis.git
   ```

3. 进入 Redis 源码目录:
   ```
   cd redis
   ```

4. 创建一个名为 `Dockerfile` 的文件,并添加以下内容:
   ```Dockerfile
   FROM ubuntu:latest

   RUN apt-get update && \
       apt-get install -y build-essential tcl

   COPY . /usr/src/redis
   WORKDIR /usr/src/redis

   RUN make && \
       make install

   EXPOSE 6379

   CMD ["redis-server"]
   ```

   这个 Dockerfile 的内容与之前提供的相同,只是没有指定特定的 Redis 版本,而是使用最新的源码。

5. 使用以下命令构建 Redis 镜像:
   ```
   docker build -t my-redis:latest .
   ```
   这将使用当前目录下的 Dockerfile 构建镜像,并将镜像标记为 `my-redis:latest`。

6. 构建完成后,你可以将镜像推送到远程 Docker 服务器:
   ```
   docker push my-redis:latest
   ```
   确保你已经登录到了远程 Docker 服务器,并且具有推送镜像的权限。

7. 在远程 Docker 服务器上,你可以使用以下命令运行 Redis 容器:
   ```
   docker run -d --name my-redis-container -p 6379:6379 my-redis:latest
   ```
   这将创建一个名为 `my-redis-container` 的容器,并将容器的 6379 端口映射到主机的 6379 端口。

现在,你已经成功地在 Windows 上编写了 Dockerfile,并在远程 Docker 服务器上构建和运行了最新版本的 Redis 镜像。

请注意,以上步骤假设你对 Docker 和 Dockerfile 有一定的了解,并且已经正确配置了 Windows 和远程 Docker 服务器之间的连接。如果你在过程中遇到任何问题,建议查阅 Docker 的官方文档或寻求相关的技术支持。

同样,在生产环境中,建议使用官方提供的 Redis 镜像,而不是自己构建镜像,以确保稳定性和安全性。
要在 Dockerfile 中指定数据卷和配置文件,并支持单机和集群模式,你可以对 Dockerfile 进行一些修改。以下是一个更完整的 Dockerfile 示例:

```Dockerfile
FROM ubuntu:latest

RUN apt-get update && \
    apt-get install -y build-essential tcl

COPY . /usr/src/redis
WORKDIR /usr/src/redis

RUN make && \
    make install

RUN mkdir -p /data/redis && \
    mkdir -p /usr/local/etc/redis

COPY redis.conf /usr/local/etc/redis/redis.conf

VOLUME /data/redis

EXPOSE 6379 16379

CMD ["redis-server", "/usr/local/etc/redis/redis.conf"]
```

这个 Dockerfile 做了以下修改:

1. 添加了创建 `/data/redis` 目录的命令,用于存储 Redis 的数据文件。

2. 添加了创建 `/usr/local/etc/redis` 目录的命令,用于存储 Redis 的配置文件。

3. 使用 `COPY` 命令将 `redis.conf` 配置文件复制到镜像中的 `/usr/local/etc/redis/redis.conf` 位置。你需要在 Redis 源码目录中准备好 `redis.conf` 文件。

4. 使用 `VOLUME` 命令将 `/data/redis` 目录声明为数据卷,以便在容器间共享数据。

5. 在 `EXPOSE` 命令中添加了 16379 端口,用于 Redis 集群通信。

6. 修改了 `CMD` 命令,使 Redis 服务器使用指定的配置文件 `/usr/local/etc/redis/redis.conf` 启动。

对于单机模式,你可以使用默认的 `redis.conf` 配置文件,无需进行特殊设置。

对于集群模式,你需要在 `redis.conf` 文件中进行相应的配置。以下是一些关键的配置项:

```
cluster-enabled yes
cluster-config-file /data/redis/nodes.conf
cluster-node-timeout 5000
appendonly yes
```

- `cluster-enabled yes` 启用集群模式。
- `cluster-config-file /data/redis/nodes.conf` 指定集群配置文件的路径。
- `cluster-node-timeout 5000` 设置集群节点超时时间。
- `appendonly yes` 启用 AOF 持久化。

请确保在 `redis.conf` 文件中进行了正确的集群配置。

使用以上的 Dockerfile 和配置文件,你可以构建支持单机和集群模式的 Redis 镜像。在运行容器时,可以通过 `-v` 参数将宿主机的目录挂载到容器的 `/data/redis` 目录,以便持久化数据。

例如,对于单机模式:

```
docker run -d --name redis -p 6379:6379 -v /path/to/data:/data/redis my-redis:latest
```

对于集群模式,你需要运行多个 Redis 容器,并进行集群配置。具体的集群配置步骤可以参考 Redis 官方文档。

请注意,以上示例仅提供了一个基本的 Dockerfile 和配置示例。在实际使用中,你可能需要根据具体的需求对配置进行调整和优化。