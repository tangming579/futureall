## Docker Compose



## Docker Swarm

### 基本概念

Docker Swarm，主要包含以下概念：

- Swarm

  Swarm本身就是“群”的意思，人群、蜂群。 这里就是指计算机集群（cluster），docker swarm命令可以创建、加入、离开一个集群。

- Node

  Node就是计算机节点，也可以认为是一个Docker节点。 Node分为两类：Manager和Worker。 一个Swarm至少要有一个Manager，部分管理命令只有在Manager上才能使用。 

- Stack

  Stack是一组Service，和docker-compose类似。 默认情况下，一个Stack共用一个Network，相互可访问，与其它Stack网络隔绝。 这个概念只是为了编排的方便。 docker stack命令可以方便地操作一个Stack，而不用一个一个地操作Service。

- Service

  Service（服务）是一类容器。 对用户来说，Service就是与Swarm交互的最核心内容。 Service有两种运行模式，一是replicated，按照一定规则在各个工作节点上运行指定个数的任务； 二是global，每个工作节点上运行一个任务。

- Task

  Task（任务）就是指运行一个容器的任务，是Swarm执行命令的最小单元。 要成功运行一个Service，需要执行一个或多个Task（取决于一个Service的容器数量）

- Load balancing

  Load balancing即负载均衡，也包含反向代理。 Swarm使用的是Ingress形式的负载均衡，即访问每个节点的某个Published端口，都可自动代理到真正的服务。

### 步骤

#### 1.  创建集群

```sh
 #创建新的Swarm
 docker swarm init --advertise-addr <MANAGER-IP>
```

- --advertise-addr 用于指定其他节点可访问的ip

- 使用docker info 可以查看当前 Swarm 状态
- 使用 docker node ls 可以查看节点信息

#### 2. 加入集群

```sh
#在manager节点上执行，可查看其他节点如何加入swarm
docker swarm join-token worker
docker swarm join-token manager
```

#### 3. 部署服务到 Swarm 中

```sh
docker service create --name helloworld --mode replicated --replicas 3 alpine
docker service create --name helloworld --mode global alpine
```

-  `docker service create` ：创建服务
- `--name` ：服务的名字为 `helloworld`.
- `--replicas` ：指定节点上运行的任务数
-  `alpine`： alpine是一个非常小的 Docker 镜像



> replicated services 按照一定规则在各个工作节点上运行指定个数的任务。
> global services 每个工作节点上运行一个任务
> 两种模式通过 docker service create 的 --mode 参数指定。

```sh
#查看运行的服务
docker service ls
```

#### 4. 在 Swarm 中查看服务

```sh
docker service inspect --pretty helloworld
#如果想通过json格式查看服务详情，使用上面的命令，去掉--pretty
docker service inspect helloworld
```

#### 5. 改变服务的伸缩性

容器运行的 service 被叫做 “tasks”，以下命令可以使服务进行弹性伸缩

```sh
#docker service scale <SERVICE-ID>=<NUMBER-OF-TASKS>
docker service scale helloworld=5
#查看任务列表
docker service ps helloworld

NAME                                    IMAGE   NODE      DESIRED STATE  CURRENT STATE
helloworld.1.8p1vev3fq5zm0mi8g0as41w35  alpine  worker2   Running        Running 7 minutes
helloworld.2.c7a7tcdq5s0uk3qr88mf8xco6  alpine  worker1   Running        Running 24 seconds
helloworld.3.6crl09vdcalvtfehfh69ogfb1  alpine  worker1   Running        Running 24 seconds
helloworld.4.auky6trawmdlcne8ad8phb0f1  alpine  manager1  Running        Running 24 seconds
helloworld.5.ba19kca06l18zujfwxyc5lkyn  alpine  worker2   Running        Running 24 seconds
```

可以使用 docker ps 命令在各个节点上查看是否有运行的容器

#### 6. 删除 swarm 服务

```sh
docker service rm helloworld
#查看 service 是否已删除
docker service inspect helloworld
```

#### 7. 滚动更新 swarm 服务

1. 以 Redis 3.0.6 为基础部署服务，然后将服务升级为 Redis 3.0.7

2. 部署 Redis 服务，配置10 秒升级延迟

   ```sh
    docker service create \
     --replicas 3 \
     --name redis \
     --update-delay 10s \
     redis:3.0.6
   
   0u6a4s31ybk7yw2wyvtikmu50
   ```

3. 查看 Redis 服务

   ```
   docker service inspect --pretty redis
   ```

4. 升级 Redis 服务

   ```
   docker service update --image redis:3.0.7 redis
   ```

   The scheduler applies rolling updates as follows by default:

   - Stop the first task.
   - Schedule update for the stopped task.
   - Start the container for the updated task.
   - If the update to a task returns `RUNNING`, wait for the specified delay period then start the next task.
   - If, at any time during the update, a task returns `FAILED`, pause the update.

5. 运行  ` docker service inspect --pretty` redis
6. 运行  `docker service ps <SERVICE-ID>`

## DockerUI

Portainer