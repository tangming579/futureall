## Docker Swarm

### 基本概念

Docker Swarm，主要包含以下概念：

- Swarm
- Node
- Stack
- Service
- Task
- Load balancing

Node就是计算机节点，也可以认为是一个Docker节点。 Node分为两类：Manager和Worker。 一个Swarm至少要有一个Manager，部分管理命令只有在Manager上才能使用。 两类Node都可以运行Service，但只有Manager上才能执行运行命令。 比如，在Manager才能使用docker node命令可以查看、配置、删除Node。

### 步骤

1. 创建集群

```sh
 #创建新的Swarm
 docker swarm init --advertise-addr <MANAGER-IP>
```

- --advertise-addr 用于指定其他节点可访问的ip

- 使用docker info 可以查看当前 Swarm 状态
- 使用 docker node ls 可以查看节点信息

2. 加入集群

```sh
#在manager节点上执行，可查看其他节点如何加入swarm
docker swarm join-token worker
docker swarm join-token manager
```

3. 部署服务到 Swarm 中

```sh
docker service create --replicas 1 --name helloworld alpine ping docker.com
```

-  `docker service create` ：创建服务
- `--name` ：服务的名字为 `helloworld`.
- `--replicas` ：指定节点上运行的任务数
-  `alpine ping docker.com`： alpine是一个非常小的 Docker 镜像，定义容器执行的命令是 ping `docker.com`

```sh
 #查看运行的服务
 docker service ls
```

