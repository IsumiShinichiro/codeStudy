从其他机器无法连接到Redis容器,通常有以下几个原因:

1. 容器端口映射问题:在启动容器时需要正确配置端口映射,确保宿主机的端口正确映射到容器的端口。

2. 防火墙问题:宿主机或网络中的防火墙可能会阻止访问映射的端口,需要开放相应的端口。

3. Redis绑定地址问题:默认情况下,Redis只接受来自127.0.0.1的连接。需要修改Redis配置文件,让其接受来自其他IP的连接。

下面是一个创建带用户名密码的Redis容器的示例,并将其端口映射到宿主机的6370端口:

1. 创建一个Redis配置文件,例如redis.conf:
```
bind 0.0.0.0
port 6379
requirepass yourpassword
```
这个配置文件指定Redis接受来自任意IP的连接,并设置了访问密码为"yourpassword"。

2. 启动Redis容器,挂载配置文件并设置端口映射:
```bash
docker run -d --name redis \
-p 6370:6379 \
-v /path/to/redis.conf:/usr/local/etc/redis/redis.conf \
redis:latest redis-server /usr/local/etc/redis/redis.conf
```
这个命令启动一个名为"redis"的容器,将宿主机的6370端口映射到容器的6379端口,并挂载了之前创建的redis.conf配置文件。

3. 验证连接:
```bash
# 在宿主机上连接Redis
redis-cli -p 6370
```

现在从宿主机应该可以连接到Redis了。输入之前设置的密码"yourpassword"进行身份验证:
```
auth yourpassword
```

4. 从其他机器连接Redis:
```bash
redis-cli -h <host_ip> -p 6370
```
将`<host_ip>`替换为运行Redis容器的宿主机的IP地址。同样,需要输入密码进行身份验证。

请注意,在生产环境中,你应该始终使用强密码,并通过SSL/TLS加密Redis的通信。同时,确保你的Redis不直接暴露在互联网上,而是放在防火墙后面,或者通过VPN访问。

希望这个示例可以帮助你解决连接问题。如果你还有任何疑问,欢迎随时问我。下面是使用Docker安装Redis集群以及使用Kubernetes部署Redis集群的步骤:

一. 使用Docker安装Redis集群

1. 下载Redis镜像
```bash
docker pull redis
```

2. 创建Redis配置文件，例如redis-cluster.tmpl
```
port ${PORT}
protected-mode no
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 192.168.0.1${NODE}
cluster-announce-port ${PORT}
cluster-announce-bus-port 1${PORT}
appendonly yes
```

3. 使用脚本创建6个Redis实例
```bash
for i in `seq 1 6`; do 
  port=$(expr 7000 + $i)
  mkdir -p ./redis-cluster/$port/conf 
  PORT=${port} NODE=${i} envsubst < ./redis-cluster.tmpl > ./redis-cluster/$port/conf/redis.conf
  docker run -d --restart always --name redis-${port} --net host \
  -v $PWD/redis-cluster/$port/conf/redis.conf:/etc/redis/redis.conf \
  -v $PWD/redis-cluster/$port/data:/data \
  redis redis-server /etc/redis/redis.conf
done
```

4. 创建集群
```bash
docker exec -it redis-7001 redis-cli --cluster create 192.168.0.11:7001 192.168.0.12:7002 192.168.0.13:7003 192.168.0.14:7004 192.168.0.15:7005 192.168.0.16:7006 --cluster-replicas 1
```

5. 验证集群
```bash
docker exec -it redis-7001 redis-cli -c
cluster info
cluster nodes
```

6. 将配置文件和数据目录打包，上传到github
```bash
tar zcvf redis-cluster.tar.gz ./redis-cluster
```

二. 使用Kubernetes部署Redis集群 

1. 创建headless service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-cluster-service
spec:
  type: ClusterIP
  clusterIP: None
  selector:
    app: redis-cluster
  ports:
  - port: 6379
    name: client
  - port: 16379 
    name: gossip
```

2. 创建ConfigMap保存redis配置
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-cluster-configmap
data:
  redis-cluster.conf: |
    cluster-enabled yes
    cluster-require-full-coverage no
    cluster-node-timeout 15000
    cluster-config-file /data/nodes.conf
    cluster-migration-barrier 1
    appendonly yes
    protected-mode no
```

3. 创建StatefulSet部署Redis
```yaml 
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-cluster
spec:
  serviceName: redis-cluster-service
  replicas: 6
  selector:
    matchLabels:
      app: redis-cluster
  template:
    metadata:
      labels:
        app: redis-cluster
    spec:
      containers:
      - name: redis
        image: redis:latest
        ports:
        - containerPort: 6379
          name: client
        - containerPort: 16379
          name: gossip
        command: ["/bin/sh"]
        args: ["-c", "redis-server /etc/redis/redis-cluster.conf"]
        readinessProbe:
          periodSeconds: 5
          tcpSocket:
            port: 6379
        livenessProbe:
          periodSeconds: 10
          tcpSocket:
            port: 6379
        volumeMounts:
        - name: conf
          mountPath: /etc/redis/
          readOnly: false
        - name: data
          mountPath: /data
          readOnly: false
      volumes:
      - name: conf
        configMap:
          name: redis-cluster-configmap
          items:
            - key: redis-cluster.conf
              path: redis-cluster.conf
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

4. 初始化Redis集群
```bash
kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 \
$(kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 ')  
```

这样就通过Docker和Kubernetes分别部署了Redis集群。关键是理解Redis集群的配置，以及如何在容器化环境下管理有状态应用。希望对你有所帮助。如有任何其他问题，欢迎随时提出。
