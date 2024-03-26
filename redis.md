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