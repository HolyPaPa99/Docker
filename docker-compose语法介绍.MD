## 一. 安装docker-comopse

**参考：**https://docs.docker.com/compose/install/

```shell
#centos
sudo yum install docker-compose-plugin
```



## 二. docker-compose服务编排

**参考：**https://docs.docker.com/compose/compose-file/compose-file-v3/

docker-compose使用yaml文件定义services（服务）、networks（网络）和volumes（挂载卷）。执行docker-compose命令会从默认路径`./docker-compose.yml`寻找compose文件。使用`.yml或.yaml`后缀名都是可以的。

docker-compose.yml例子：

```groovy
version: "3.9"
services:

  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        max_replicas_per_node: 1
        constraints:
          - "node.role==manager"

  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - "5000:80"
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure

  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - "5001:80"
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints:
          - "node.role==manager"

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints:
          - "node.role==manager"

networks:
  frontend:
  backend:

volumes:
  db-data:
```

### 1. services

#### 1.1 build

build 可以是指定为包含构建上下文路径的字符串，也可以是作为在上下文中指定路径的对象，包含可选的 Dockerfile 和 args等。如果在服务编排中指定 image 和 build，那么 Compose 会使用 image 中指定的镜像名称和可选标签名构建的镜像。如：

```groovy
version: "3.9"
services:
  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
```

```groovy
version: "3.9"
services:
  webapp:
    build: ./dir
		image: webapp:tag
```

- ***context***

  context可以是包含Dockerfile的目录的路径，也可以是git存储库的url。当提供的值是相对路径时，它被解释为相对于 Compose 文件的位置。 此目录也是发送给Docker守护程序的构建上下文。如果不指定image名称和标签名，Compose自动生成镜像名称和标签名构建并标记该镜像。

  ```groovy
  context: ./dir
  ```

  

- ***dockerfile***

  指定dockerfile文件名。

  ```groovy
  build:
    context: .
    dockerfile: Dockerfile-alternate
  ```

  

- ***args***

  添加构建参数，它们是只能在构建过程中访问的环境变量。

  ```groovy
  # syntax=docker/dockerfile:1
  
  ARG buildno
  ARG gitcommithash
  
  RUN echo "Build number: $buildno"
  RUN echo "Based on commit: $gitcommithash"
  ```

  有两种声明方式，如下：

  ```groovy
  build:
    context: .
    args:
      buildno: 1
      gitcommithash: cdc3b19
  ```

  ```groovy
  build:
    context: .
    args:
      - buildno=1
      - gitcommithash=cdc3b19
  ```

  您可以在指定构建参数时省略该值，在这种情况下，它在构建时的值是 Compose 运行环境中的值。

  ```groovy
  args:
    - buildno
    - gitcommithash
  ```

  ***注意：YAML 布尔值（“true”、“false”、“yes”、“no”、“on”、“off”）必须用引号括起来，以便解析器将它们解释为字符串。***

  

- ***cache_from***

  缓存可用镜像，用于多阶段构建。

  ```groovy
  build:
    context: .
    cache_from:
      - alpine:latest
      - corp/web_app:3.14
  ```

  

- ***labels***

  使用docker标签将元数据添加到容器，与Dockerfile中的LABELS类似。有两种添加方式，如下：

  ```groovy
  build:
    context: .
    labels:
      com.example.description: "Accounting webapp"
      com.example.department: "Finance"
      com.example.label-with-empty-value: ""
  ```

  ```groovy
  build:
    context: .
    labels:
      - "com.example.description=Accounting webapp"
      - "com.example.department=Finance"
      - "com.example.label-with-empty-value"
  ```

  

- ***network***

  在构建期间为 RUN 指令设置容器的网络连接。

  ```groovy
  build:
    context: .
    network: host
  ```

  ```groovy
  build:
    context: .
    network: custom_network_1
  ```

  ```groovy
  build:
    context: .
    network: none
  ```

  

- ***shm_size***

  为此构建的容器设置 /dev/shm 分区的大小。 指定为表示字节数的整数值或字符串。

  ```groovy
  build:
    context: .
    shm_size: '2gb'
  ```

  ```groovy
  build:
    context: .
    shm_size: 10000000
  ```

  

- ***target***

  构建 Dockerfile 中定义的指定阶段。用于多阶段构建。

  ```groovy
  build:
    context: .
    target: prod
  ```

  

#### 1.2 cap_add, cap_drop

添加或删除容器功能。

```groovy
cap_add:
  - ALL

cap_drop:
  - NET_ADMIN
  - SYS_ADMIN
```

注意：在 swarm 模式下部署堆栈时会忽略 cap_add 和 cap_drop 选项

#### 1.3 cgroup_parent

为容器指定一个可选的父 cgroup。

```groovy
cgroup_parent: m-executor-abcd
```

注意：在 swarm 模式下部署堆栈时会忽略cgroup_parent选项

#### 1.4 command

覆盖默认命令。

```groovy
command: bundle exec thin -p 3000
```

或

```groovy
command: ["bundle", "exec", "thin", "-p", "3000"]
```



#### 1.5 configs

service级别的引用配置声明。使用的配置必须已经存在或在当前compose文件的顶层configs中定义，否则部署失败`config not found`。

***注意：my_other_config 被定义为外部资源，这意味着它已经在 Docker 中定义，无论是通过运行 docker config create 命令还是通过另一个堆栈部署。***

```groovy
version: "3.9"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    configs:
      - my_config
      - my_other_config
configs:
  my_config:
    file: ./my_config.txt
  my_other_config:
    external: true
```

或：

```groovy
version: "3.9"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    configs:
      - source: my_config
        target: /redis_config
        uid: '103'
        gid: '103'
        mode: 0440
configs:
  my_config:
    file: ./my_config.txt
  my_other_config:
    external: true
```

- ***source***

  在此配置中定义的配置标识符。

- ***target***

  要挂载在服务的任务容器中的文件的路径和名称。 如果未指定，则默认为 /<source>。

- ***uid和gid***

  在服务的任务容器中拥有已挂载配置文件的UID 或 GID。 如果未指定，则两者在 Linux 上均默认为 0。 在 Windows 上不支持。

- ***mode***

  挂载在服务的任务容器中的文件的权限，以八进制表示。 例如，0444 代表世界可读。 默认值为 0444。配置不能写，因为它们被挂载在一个临时文件系统中，所以如果你设置了可写位，它会被忽略。 可执行位可以设置。 

  如果您不熟悉 UNIX 文件权限模式，可以使用权限计算器：http://permissions-calculator.org/

#### 1.6 container_name

指定自定义容器名称。

```groovy
container_name: my-web-container
```

注意：在 swarm 模式下部署时，container_name 选项被忽略

#### 1.7 credential_spec

为管理服务帐户配置凭据。 此选项仅用于使用 Windows 容器的服务。 credential_spec 必须采用 file://<filename> 或 registry://<value-name> 格式。

使用 file: 时，引用的文件必须存在于 Docker 数据目录的 CredentialSpecs 子目录中，该目录在 Windows 上默认为 C:\ProgramData\Docker\。 以下示例从名为 C:\ProgramData\Docker\CredentialSpecs\my-credential-spec.json 的文件加载凭据规范。

```groovy
credential_spec:
  file: my-credential-spec.json
```

在为服务配置 gMSA 凭证时，您只需要使用 config 指定凭证规范，如下例所示：

```groovy
version: "3.9"
services:
  myservice:
    image: myimage:latest
    credential_spec:
      config: my_credential_spec

configs:
  my_credentials_spec:
    file: ./my-credential-spec.json|
```



#### 1.8 depends_on

服务之间的依赖关系。 服务依赖会导致以下行为：

- `docker-compose up`按依赖顺序启动服务。 在下面的例子中，db 和 redis 在 web 之前启动。
- `docker-compose up 服务名称`自动包含服务的依赖项。 在下面的示例中，`docker-compose up web`会自动创建并启动`db`和`redis`。
- `docker-compose stop`按依赖顺序停止服务。 在以下示例中，`web`在`db`和`redis`之前停止。

```groovy
version: "3.9"
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

***使用depends_on时需要注意以下几点：***

- 在启动 web 之前，depends_on 不会等待 db 和 redis “准备好” - 只在它们启动之前。 如果您需要等待服务准备就绪，请参阅控制启动顺序以了解有关此问题的更多信息以及解决该问题的策略。
- 在使用版本 3 Compose 文件以 swarm 模式部署堆栈时，将忽略depends_on 选项。

#### 1.9 deploy

指定与服务部署和运行相关的配置。 这仅在使用`docker stack deploy`部署到`swarm`时生效，并且被`docker-compose up`和`docker-compose run`忽略。

```groovy
version: "3.9"
services:
  redis:
    image: redis:alpine
    deploy:
      replicas: 6
      placement:
        max_replicas_per_node: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
```

- ***endpoint_mode***

  为连接到 swarm 的外部客户端指定服务发现方法。

  - `endpoint_mode: vip`

    Docker 为服务分配一个虚拟 IP (VIP)，作为客户端访问网络上的服务的前端。 Docker 在客户端和服务的可用工作节点之间路由请求，而客户端不知道有多少节点参与服务或其 IP 地址或端口。 （这是默认设置。）

  - `endpoint_mode: dnsrr`

    DNS round-robin (DNSRR) 服务发现不使用单个虚拟 IP。 Docker 为服务设置 DNS 条目，以便对服务名称的 DNS 查询返回 IP 地址列表，并且客户端直接连接到其中之一。 如果您想使用自己的负载均衡器，或者对于混合 Windows 和 Linux 应用程序，DNSRR非常有用。

  ```groovy
  version: "3.9"
  
  services:
    wordpress:
      image: wordpress
      ports:
        - "8080:80"
      networks:
        - overlay
      deploy:
        mode: replicated
        replicas: 2
        endpoint_mode: vip
  
    mysql:
      image: mysql
      volumes:
         - db-data:/var/lib/mysql/data
      networks:
         - overlay
      deploy:
        mode: replicated
        replicas: 2
        endpoint_mode: dnsrr
  
  volumes:
    db-data:
  
  networks:
    overlay:
  ```

  

- ***labels***

  指定服务的元数据。 这些元数据只设置在服务上，而不设置在服务的容器上。

  ```groovy
  version: "3.9"
  services:
    web:
      image: web
      deploy:
        labels:
          com.example.description: "This label will appear on the web service"
  ```

  

- ***mode***

  `global` : 一个swarm节点一个容器

  `replicated` : 复制模式需指定容器数量， 默认。

  ```groovy
  version: "3.9"
  services:
    worker:
      image: dockersamples/examplevotingapp_worker
      deploy:
        mode: global
  ```

  

- ***placement***

  指定约束和首选项。

  详细设置项参考：

  https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints---constraint

  https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-placement-preferences---placement-pref

  ```groovy
  version: "3.9"
  services:
    db:
      image: postgres
      deploy:
        placement:
          constraints:
            - "node.role==manager"
            - "engine.labels.operatingsystem==ubuntu 18.04"
          preferences:
            - spread: node.labels.zone
  ```

  

- ***max_replicas_per_node***

  当服务在`replicated`模式下，设置每个节点可以复制数量的上限。

  当指定容器数量超出所有运行节点负荷时会抛出错误：no suitable node (max replicas per node limit exceed)`

  ```groovy
  version: "3.9"
  services:
    worker:
      image: dockersamples/examplevotingapp_worker
      networks:
        - frontend
        - backend
      deploy:
        mode: replicated
        replicas: 6
        placement:
          max_replicas_per_node: 1
  ```

  

- ***replicas***

  当服务在`replicated`模式下，设置运行容器数量。

  ```groovy
  version: "3.9"
  services:
    worker:
      image: dockersamples/examplevotingapp_worker
      networks:
        - frontend
        - backend
      deploy:
        mode: replicated
        replicas: 6
  ```

  

- ***resources***

  容器资源限制配置。

  在下面示例中，redis 服务被限制为使用不超过 50M 的内存和 0.50（单核的 50%）的可用处理时间（CPU），并保留 20M 的内存和 0.25 个 CPU 时间始终可用。

  ```groovy
  version: "3.9"
  services:
    redis:
      image: redis:alpine
      deploy:
        resources:
          limits:
            cpus: '0.50'
            memory: 50M
          reservations:
            cpus: '0.25'
            memory: 20M
  ```

  如果您的服务或容器尝试使用比系统可用的更多内存，您可能会遇到内存不足异常 (OOME)，并且容器或 Docker 守护程序可能会被内核 OOM 杀手杀死。 为防止发生这种情况，请确保您的应用程序在具有足够内存的主机上运行，并参阅***了解内存不足的风险***：https://docs.docker.com/config/containers/resource_constraints/#understand-the-risks-of-running-out-of-memory。

- ***restart_policy***

  配置容器退出时的重启策略。

  - condition

    可选值：`none`,`on-failure`和`any`(默认)

  - delay

    重启尝试间隔时间，默认`5s`。

  - max_attempts

    最大重启尝试次数，默认不断尝试。

  - window

    判断重启是否成功等待时间，默认立即执行判断。

  ```groovy
  version: "3.9"
  services:
    redis:
      image: redis:alpine
      deploy:
        restart_policy:
          condition: on-failure
          delay: 5s
          max_attempts: 3
          window: 120s
  ```

  

- ***rollback_config***

  配置在更新失败的情况下应如何回滚服务。

  - `parallelism`

    一次回滚的容器数量。 如果设置为 0，则所有容器同时回滚。

  - `delay`

  - 每个容器组回滚之间等待的时间（默认为 0 秒）。

  - `failure_action`

  - 如果回滚失败该怎么办。 `continue` 或 `pause` （默认为 `pause`）

  - `monitor`

    每个任务更新后监控失败的持续时间`(ns|us|ms|s|m|h)`（默认5s）**注意**：设置为0将使用默认5s。

  - `max_failure_ratio`

    回滚期间允许的故障率（默认为 0）。

  - `order`

    回滚期间的操作顺序。 `stop-first`（旧任务在开始新任务之前停止）或`start-first`（首先启动新任务，并且正在运行的任务短暂重叠）之一（默认为`stop-first`）。

- ***update_config***

  配置应如何更新服务， 用于配置滚动更新。

  - `parallelism`

     一次更新的容器数量。

  - `delay`

    在更新一组容器之间等待的时间。

  - `failure_action`

    如果更新失败该怎么办。 `continue`、`rollback` 或 `pause`（默认值：`pause`）。

  - `monitor`

     每个任务更新后监控失败的持续时间`(ns|us|ms|s|m|h)`（默认5s）**注意**：设置为0将使用默认5s。

  - `max_failure_ratio`

    更新期间允许的故障率。

  - `order`

    更新期间的操作顺序。 `stop-first`（旧任务在启动新任务之前停止）或`start-first`（新任务首先启动，并且正在运行的任务短暂重叠）之一（默认为`stop-first`）**注意* *：仅支持 v3.4 及更高版本。

  ```groovy
  version: "3.9"
  services:
    vote:
      image: dockersamples/examplevotingapp_vote:before
      depends_on:
        - redis
      deploy:
        replicas: 2
        update_config:
          parallelism: 2
          delay: 10s
          order: stop-first
  ```

- docker stack部署不支持选项

  下列子选项 (支持`docker-compose up` 和 `docker-compose run`) 不支持`docker stack deploy` 或 `deploy` key：

  - build

  - cgroup_parent

  - container_name

  - devices

  - tmpfs

  - external_links

  - links

  - network_mode

  - restart

  - security_opt

  - userns_mode

    

#### 1.10 devices

设备映射列表。 使用与 --device docker 客户端创建选项相同的格式。

```groovy
devices:
  - "/dev/ttyUSB0:/dev/ttyUSB0"
```

在集群模式下部署stack时，设备选项被忽略

#### 1.11 dns

自定义 DNS 服务器。 可以是单个值或列表。

```groovy
dns: 8.8.8.8
```

```groovy
dns:
  - 8.8.8.8
  - 9.9.9.9
```



#### 1.12 dns_search

自定义 DNS 搜索域。 可以是单个值或列表。

```groovy
dns_search: example.com
```

```groovy
dns_search:
  - dc1.example.com
  - dc2.example.com
```



#### 1.13 entrypoint

覆盖默认entrypoint。设置`entrypoint`会覆盖镜像Dockerfile里设置的`ENTRYPOINT`和`CMD`指令。

```groovy
entrypoint: /code/entrypoint.sh
```

```groovy
entrypoint: ["php", "-d", "memory_limit=-1", "vendor/bin/phpunit"]
```



#### 1.14 env_file

从文件添加环境变量。 可以是单个值或列表。如果您使用 `docker-compose -f FILE` 指定了 Compose 文件，则 env_file` 中的路径相对于该文件所在的目录。在 `environment` 部分中声明的环境变量会覆盖这些值——即使值是空的或未定义的，也是如此。

```groovy
env_file: .env
```

```groovy
env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/runtime_opts.env
```



#### 1.15 environment

添加环境变量。 您可以使用数组或字典。 任何布尔值（true、false、yes、no）都需要用引号括起来，以确保它们不会被 YML 解析器转换为 True 或 False。只有一个键的环境变量被解析为它们在运行 Compose 的机器上的值，这适用于密钥或特定主机的值设置。

```groovy
environment:
  RACK_ENV: development
  SHOW: 'true'
  SESSION_SECRET:
```

```groovy
environment:
  - RACK_ENV=development
  - SHOW=true
  - SESSION_SECRET
```



#### 1.16 expose

开放内部端口而不将它们发布到主机——它们只能被链接的服务访问。

```groovy
expose:
  - "3000"
  - "8000"
```



#### 1.17 external_links

链接到在此 docker-compose.yml 之外甚至 Compose 之外启动的容器，尤其是对于提供共享或公共服务的容器。 在指定容器名称和链接别名 (CONTAINER:ALIAS) 时，external_links 遵循类似于旧选项链接的语义。外部创建的容器必须至少连接到与链接到它们的服务相同的网络之一。 链接是一个遗留选项。 我们建议改用网络。

```groovy
external_links:
  - redis_1
  - project_db_1:mysql
  - project_db_1:postgresql
```

在群模式下部署堆栈时，external_links 选项被忽略



#### 1.18 extra_hosts

添加主机名映射。 使用与 docker 客户端 --add-host 参数相同的值。

```groovy
extra_hosts:
  - "somehost:162.242.195.82"
  - "otherhost:50.31.209.229"
```

此服务的容器内的 /etc/hosts 中创建一个包含 ip 地址和主机名的条目，例如：

```groovy
162.242.195.82	somehost
50.31.209.229		otherhost
```



#### 1.19 healthcheck

配置运行的检查以确定此服务的容器是否“健康”。 有关健康检查如何工作的详细信息，请参阅 HEALTHCHECK Dockerfile 指令的文档。

https://docs.docker.com/engine/reference/builder/#healthcheck

```groovy
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
  start_period: 40s
```

`test` 必须是字符串或列表。 如果是列表，则第一项必须是 `NONE`、`CMD` 或 `CMD-SHELL`。 如果是字符串，则相当于指定 `CMD-SHELL` 后跟该字符串。

```groovy
# Hit the local web app
test: ["CMD", "curl", "-f", "http://localhost"]
```

```groovy
test: ["CMD-SHELL", "curl -f http://localhost || exit 1"]
```

```groovy
test: curl -f https://localhost || exit 1
```

要禁用镜像设置的任何默认健康检查，您可以使用 `disable: true`。 这相当于指定 `test: ["NONE"]`。

```groovy
healthcheck:
  disable: true
```



#### 1.20 image

指定启动容器的镜像。 可以是存储库/标签或部分图像 ID。

```groovy
image: redis
```

```groovy
image: ubuntu:18.04
```

```groovy
image: tutum/influxdb
```

```groovy
image: example-registry.com:4000/postgresql
```

```groovy
image: a4bc65fd
```

如果镜像不存在，Compose 会尝试拉取它，除非您还指定了 build，在这种情况下，它会使用指定的选项构建它并使用指定的标签对其进行标记。

#### 1.21 init

在容器内运行一个 init 来转发信号和获取进程。 将此选项设置为 true 可为服务启用此功能。
默认使用的 init 二进制文件是 Tini，安装在守护程序主机上的 /usr/libexec/docker-init 中。 您可以通过 init-path 配置选项将守护进程配置为使用自定义 init 二进制文件。

```groovy
version: "3.9"
services:
  web:
    image: alpine:latest
    init: true
```



#### 1.22 isolation

指定容器的隔离技术。 在 Linux 上，唯一支持的值是默认值。 在 Windows 上，可接受的值为 default、process 和 hyperv。

#### 1.23 labels

使用 Docker 标签向容器添加元数据。 您可以使用数组或字典。建议您使用反向 DNS 表示法，以防止您的标签与其他软件使用的标签冲突。

```groovy
labels:
  com.example.description: "Accounting webapp"
  com.example.department: "Finance"
  com.example.label-with-empty-value: ""
```

```groovy
labels:
  - "com.example.description=Accounting webapp"
  - "com.example.department=Finance"
  - "com.example.label-with-empty-value"
```



#### 1.24 links

链接到另一个服务中的容器。 指定服务名称和链接别名（“SERVICE:ALIAS”），或仅指定服务名称。以弃用，建议使用networks。

```groovy
web:
  links:
    - "db"
    - "db:database"
    - "redis"
```



#### 1.25 logging

服务的日志配置。

```groovy
logging:
  driver: syslog
  options:
    syslog-address: "tcp://192.168.0.42:123"
```

Driver指定了服务容器的日志驱动程序，就像 docker run 的 --log-driver 选项一样(https://docs.docker.com/config/containers/logging/configure/)。默认值`json-file`。

```groovy
driver: "json-file"
```

```groovy
driver: "syslog"
```

```groovy
driver: "none"
```

```groovy
version: "3.9"
services:
  some-service:
    image: some-service
    logging:
      driver: "json-file"
      options:
        max-size: "200k"
        max-file: "10"
```



#### 1.26 network_mode

网络模式。 使用与 docker client --network 参数相同的值，加上特殊形式 service:[service name]。

```groovy
network_mode: "bridge"
```
```groovy
network_mode: "host"
```
```groovy
network_mode: "none"
```
```groovy
network_mode: "service:[service name]"
```
```groovy
network_mode: "container:[container name/id]"
```

#### 1.27 networks

设置要加入的网络，引用当前compose顶级networks下设置网络。

```groovy
services:
  some-service:
    networks:
     - some-network
     - other-network
```

- ***aliases***

  网络上此服务的别名（替代主机名）。 同一网络上的其他容器可以使用服务名称或此别名连接到服务的容器之一。由于别名是网络范围的，因此相同的服务在不同的网络上可以有不同的别名。

  ```groovy
  version: "3.9"
  
  services:
    web:
      image: "nginx:alpine"
      networks:
        - new
  
    worker:
      image: "my-worker-image:latest"
      networks:
        - legacy
  
    db:
      image: mysql
      networks:
        new:
          aliases:
            - database
        legacy:
          aliases:
            - mysql
  
  networks:
    new:
    legacy:
  ```

  

- #### ***ipv4_address, ipv6_address***

  加入网络时，为此服务指定容器的静态 IP 地址。顶级网络部分中的相应网络配置必须有一个 ipam 块，其中包含覆盖每个静态地址的子网配置。

  ```groovy
  version: "3.9"
  
  services:
    app:
      image: nginx:alpine
      networks:
        app_net:
          ipv4_address: 172.16.238.10
          ipv6_address: 2001:3984:3989::10
  
  networks:
    app_net:
      ipam:
        driver: default
        config:
          - subnet: "172.16.238.0/24"
          - subnet: "2001:3984:3989::/64"
  ```

  

#### 1.28 pid

将 PID 模式设置为主机 PID 模式。 这会在容器和主机操作系统之间共享 PID 地址空间。 使用此标志启动的容器可以访问和操作裸机命名空间中的其他容器，反之亦然。

```groovy
pid: "host"
```



#### 1.29 ports

开放指定端口。

短格式语法：

- `HOSTPORT:CONTAINERPORT`

- `CONTAINERPORT`

  只声明容器端口 (运行时会临时选择一个可用主机端口).

- `IPADDR:HOSTPORT:CONTAINERPORT`

  默认IPADDR是 0.0.0.0。如果HOSTPORT为空则运行时会临时选择一个可用主机端口。

注意：当以 HOST:CONTAINER 格式映射端口时，当使用低于 60 的容器端口时，您可能会遇到错误的结果，因为 YAML 将 xx:yy 格式的数字解析为 base-60 值。 因此，我们建议始终将您的端口映射明确指定为字符串。

```groovy
ports:
  - "3000"
  - "3000-3005"
  - "8000:8000"
  - "9090-9091:8080-8081"
  - "49100:22"
  - "127.0.0.1:8001:8001"
  - "127.0.0.1:5000-5010:5000-5010"
  - "127.0.0.1::5000"
  - "6060:6060/udp"
  - "12400-12500:1240"
```



长格式语法：

长格式语法允许配置不能用短格式表达的附加字段。

- `target`

  容器端口

- `published`

  开放端口

- `protocol

  `tcp` or `udp`

- `mode`

  `host` - 表示发布主机端口

  `ingress` - 表示集群负载均衡端口

```groovy
ports:
  - target: 80
    published: 8080
    protocol: tcp
    mode: host
```



#### 1.30 profiles

为服务设定`profile`标签。指定profile标签的服务只有在给定的profile被激活时在会启动，未指定profile的服务一直会启动。

详细使用说明参考：https://docs.docker.com/compose/profiles/

```groovy
profiles: ["frontend", "debug"]
```

```groovy
profiles:
  - frontend
  - debug

```

如下面例子，服务frontend和phpmyadmin指定里profile，在运行**`docker-compose --profile frontend --profile debug up`**时，frontend和phpmyadmin服务才被启动。

```groovy
version: "3.9"
services:
  frontend:
    image: frontend
    profiles: ["frontend"]

  phpmyadmin:
    image: phpmyadmin
    depends_on:
      - db
    profiles:
      - debug

  backend:
    image: backend

  db:
    image: mysql
```

#### 1.31 restart

重启策略。

- `no` 

  默认的重启策略，在任何情况下都不会重启容器。

- `always` 

  容器始终重新启动。

- `on-failure` 

  当exit code表示容器出现on-failure错误时，容器会重新启动。

- `unless-stopped`

  除非容器停止（手动或以其他方式），否则总是重新启动容器。

#### 1.32 secrets

授权在服务容器使用secret数据。

关于 docker secret管理参考：https://docs.docker.com/engine/swarm/secrets/

短格式语法：

这会授予容器访问secret的权限，并将其挂载在容器内的 /run/secrets/<secret_name> 中。 源名称和目标挂载点都设置为secret名称。

以下示例使用短格式语法授予 redis 服务访问 my_secret 和 my_other_secret 的权限。 my_secret 的值被设置为文件 ./my_secret.txt 的内容，而 my_other_secret 被定义为外部资源，这意味着它已经在 Docker 中被定义，要么通过运行 `docker secret create` 命令创建，要么通过另一个堆栈部署。 如果外部机密不存在，堆栈部署将失败并出现`secret not found`错误。

```groovy
version: "3.9"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    secrets:
      - my_secret
      - my_other_secret
secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```

长格式语法：

- `source`

  secret唯一标识

- `target`

  挂载在服务容器 `/run/secrets/` 中的文件的名称。 如果未指定，则默认为 `source`。

- `uid` and `gid`

  拥有服务容器 `/run/secrets/` 中的文件的 UID 或 GID。 如果未指定，两者都默认为“0”。

- `mode`

  挂载在服务容器的 `/run/secrets/` 中的文件的权限，采用八进制表示法。 例如，“0444”代表世界可读。 Docker 1.13.1 中的默认值为“0000”，但在较新版本中为“0444”。 Secrets 不能写，因为它们被挂载在一个临时文件系统中，所以如果你设置了 writable 位，它会被忽略。 可执行位可以设置。 

  如果您不熟悉 UNIX 文件权限模式，可以使用权限计算器：http://permissions-calculator.org/

```groovy
version: "3.9"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    secrets:
      - source: my_secret
        target: redis_secret
        uid: '103'
        gid: '103'
        mode: 0440
secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```



#### 1.33 security_opt

覆盖每个容器的默认标签方案。

```groovy
security_opt:
  - label:user:USER
  - label:role:ROLE
```



#### 1.34 stop_grace_period

设置当容器接收到退出信号时，发送SIGKILL前等待容器退出时间，默认10秒。

```groovy
stop_grace_period: 1s
```

```groovy
stop_grace_period: 1m30s
```



#### 1.35 stop_signal

设置一个替代信号来停止容器。 默认情况下，`stop` 使用 SIGTERM。

```groovy
stop_signal: SIGUSR1
```

#### 1.36 sysctls

在容器中设置的内核参数。 您可以使用数组或字典。只能使用内核中命名空间的 sysctls。 Docker 不支持在容器内更改 sysctl 也会修改主机系统。 有关支持的 sysctls 的概述，请参考：https://docs.docker.com/engine/reference/commandline/run/#configure-namespaced-kernel-parameters-sysctls-at-runtime

```groovy
sysctls:
  net.core.somaxconn: 1024
  net.ipv4.tcp_syncookies: 0
```
```groovy
sysctls:
  - net.core.somaxconn=1024
  - net.ipv4.tcp_syncookies=0
```

#### 1.37 tmpfs

在容器内挂载一个临时文件系统。 可以是单个值或列表。

```groovy
tmpfs: /run
```

```groovy
tmpfs:
  - /run
  - /tmp
```

Size 参数以字节为单位指定 tmpfs 挂载的大小。 默认无限制。

```groovy
- type: tmpfs
  target: /app
  tmpfs:
    size: 1000
```

#### 1.38 ulimits

覆盖容器默认的ulimits。可以设置单个限制整数值，也可以使用soft/hard限制对。

Override the default ulimits for a container. You can either specify a single limit as an integer or soft/hard limits as a mapping.

```groovy
ulimits:
  nproc: 65535
  nofile:
    soft: 20000
    hard: 40000
```



#### 1.39 userns_mode 

禁用服务的用户命名空间。

```groovy
userns_mode: "host"
```



#### 1.40 volumes

service子选项配置，用于挂载主机路径或命名卷。

```groovy
version: "3.9"
services:
  web:
    image: nginx:alpine
    volumes:
      - type: volume
        source: mydata
        target: /data
        volume:
          nocopy: true
      - type: bind
        source: ./static
        target: /opt/app/static

  db:
    image: postgres:latest
    volumes:
      - "/var/run/postgres/postgres.sock:/var/run/postgres/postgres.sock"
      - "dbdata:/var/lib/postgresql/data"

volumes:
  mydata:
  dbdata:
```

***短语法格式***

`[SOURCE:]TARGET[:MODE]` 格式：

- `SOURCE` 

  主机文件路径或命名卷

- `TARGET` 

  容器文件路径

- `MODE`  

  挂载文件模式：`ro` 只读  `rw`  读写（默认）。

可以在主机上挂载一个相对路径，该路径相对于正在使用的 Compose 配置文件的目录。 相对路径应该总是以 `.` 或 `..` 开头。

```groovy
volumes:
  # Just specify a path and let the Engine create a volume
  - /var/lib/mysql

  # Specify an absolute path mapping
  - /opt/data:/var/lib/mysql

  # Path on the host, relative to the Compose file
  - ./cache:/tmp/cache

  # User-relative path
  - ~/configs:/etc/configs/:ro

  # Named volume
  - datavolume:/var/lib/mysql
```



***长语法格式***

长语法格式允许配置不能用短格式表达的附加字段。

- `type`

  挂载类型： `volume`, `bind`, `tmpfs` 或 `npipe`

- `source`

  挂载的来源、主机上用于绑定挂载的路径或在顶级`volumes` 中定义的卷的名称。不适用于 tmpfs 挂载。

- `target`

  挂载指向的容器路径。

- `read_only`

  挂载卷只读标志。

- `bind`
  
  配置额外的挂载选项

  - `propagation`: 绑定传播模式

- `volume`
  
  配置额外的卷选项

  - `nocopy`: 创建卷时禁用从容器复制数据的标志
- `tmpfs`
  
  :配置额外的 tmpfs 选项
  
  - `size`: tmpfs 挂载的大小（以字节为单位）

```groovy
version: "3.9"
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - type: volume
        source: mydata
        target: /data
        volume:
          nocopy: true
      - type: bind
        source: ./static
        target: /opt/app/static

networks:
  webnet:

volumes:
  mydata:
```



### 2. networks

顶层networks声明定义关键字，声明将会被创建的docker网络。

- driver

  网络驱动，默认驱动取决于docker引擎配置，一般情况下单主机为`bridge`，swarm集群为`overlay`。

  `bridge` - 一般情况下单主机默认驱动为`bridge`。

  `overlay` - 在swarm集群里创建连接多节点的网络。

  `host` - 使用主机网络，只支持`docker stack`命令使用，compose建议使用`network_mode`配置。

  `none` - 无网络，只支持`docker stack`命令使用，compose建议使用`network_mode`配置。

  ```groovy
  version: "3.9"
  services:
    web:
      networks:
        hostnet: {}
  
  networks:
    hostnet:
      external: true
      name: host
  ```

  

- dirver_opts

  驱动可选参数，使用键值对列表方式配置。

  ```groovy
  driver_opts:
    foo: "bar"
    baz: 1
  ```

- attachable

  仅当网络驱动是`overlay`时可用。为`true`时容器可以附加到该网络，与多个服务之间通信，也可以附加到其他集群网络。

  ```groovy
  networks:
    mynet1:
      driver: overlay
      attachable: true
  ```

- enable_ipv6

  启用ipv6。

- ipam

  自定义IPAM. 可选参数:

  - `driver`

    自定义驱动。

  - `config`

    多个配置块列表，每个配置块都包含`subnet`。

    - `subnet`: CIDR格式代表一个网络ip域。

  例子:

  ```groovy
  ipam:
    driver: default
    config:
      - subnet: 172.28.0.0/16
  ```

- internal

  默认情况下，Docker 还会将桥接网络连接到它以提供外部连接。 如果要创建与外部隔离的覆盖网络，可以将此选项设置为`true`。

- labels

  使用docker标签向容器添加元数据，可以使用数组或字典。

  建议您使用反向 DNS 表示法，以防止您的标签与其他软件使用的标签冲突。

  ```groovy
  labels:
    com.example.description: "Financial transaction network"
    com.example.department: "Finance"
    com.example.label-with-empty-value: ""
  ```

  ```groovy
  labels:
    - "com.example.description=Financial transaction network"
    - "com.example.department=Finance"
    - "com.example.label-with-empty-value"
  ```

- external

  如果设置为`true`，则指定此网络是在 Compose 之外创建的。 `docker-compose up` 不会尝试创建它，如果它不存在就会引发错误。

  在下面的示例中，`proxy` 是通往外部世界的门户。 Compose 没有尝试创建一个名为 `[projectname]_outside` 的网络，而是查找一个名为 `outside` 的现有网络，并将 `proxy` 服务的容器连接到它。

  ```groovy
  version: "3.9"
  
  services:
    proxy:
      build: ./proxy
      networks:
        - outside
        - default
    app:
      build: ./app
      networks:
        - default
  
  networks:
    outside:
      external: true
  ```

  您还可以将网络名称与在 Compose 文件中用于引用它的名称分开指定:

  ```groovy
  version: "3.9"
  networks:
    outside:
      external:
        name: actual-name-of-network
  ```

- name

  自定义网络名称。

  ```groovy
  version: "3.9"
  networks:
    network1:
      name: my-app-net
  ```

  也可以和 `external` 属性一起使用:

  ```groovy
  version: "3.9"
  networks:
    network1:
      external: true
      name: my-app-net
  ```

### 3. volumes

创建命名卷。可以在多个service中引用被创建的命名卷。

例子：

```groovy
version: "3.9"

services:
  db:
    image: db
    volumes:
      - data-volume:/var/lib/db
  backup:
    image: backup-service
    volumes:
      - data-volume:/var/lib/backup/data

volumes:
  data-volume:
```

顶级 `volumes` 键下的条目可以为空，在这种情况下，它使用引擎配置的默认驱动程序（在大多数情况下，就是 `local` 驱动程序）。 或者，您可以使用以下键对其进行配置：

- driver

  指定该卷应使用哪个卷驱动程序。 默认为 Docker 引擎配置使用的任何驱动程序，在大多数情况下是`local`。 如果驱动程序不可用，当`docker-compose up` 尝试创建卷时，引擎会返回错误。

  ```groovy
  driver: foobar
  ```

  

- driver_opts

  将可选项列表指定为键值对以传递给此卷的驱动程序。 这些选项取决于所使用的驱动程序 - 有关更多信息，请参阅驱动程序的文档。 

  ```groovy
  volumes:
    example:
      driver_opts:
        type: "nfs"
        o: "addr=10.40.0.199,nolock,soft,rw"
        device: ":/docker/example"
  ```

- external

  如果设置为`true`，则指定该卷是在 Compose 之外创建的。 `docker-compose up` 不会尝试创建它，如果它不存在就会引发错误。

  在下面的示例中，Compose 没有尝试创建一个名为 `[projectname]_data` 的卷，而是查找一个名为 `data` 的现有卷并将其挂载到 `db` 服务的容器中。

  ```groovy
  version: "3.9"
  
  services:
    db:
      image: postgres
      volumes:
        - data:/var/lib/postgresql/data
  
  volumes:
    data:
      external: true
  ```

  您还可以将卷的名称与在 Compose 文件中用于引用它的名称分开指定：

  ```groovy
  volumes:
    data:
      external:
        name: actual-name-of-volume
  ```

- labels

  使用 Docker 标签向容器添加元数据，可以使用数组或字典。

  建议您使用反向 DNS 表示法，以防止您的标签与其他软件使用的标签冲突。

  ```groovy
  labels:
    com.example.description: "Database volume"
    com.example.department: "IT/Ops"
    com.example.label-with-empty-value: ""
  ```

  ```groovy
  labels:
    - "com.example.description=Database volume"
    - "com.example.department=IT/Ops"
    - "com.example.label-with-empty-value"
  ```

- name

为此卷设置自定义名称。 名称字段可用于引用包含特殊字符的卷。 该名称按原样使用，并且**不** 限定为堆栈名称。

```groovy
version: "3.9"
volumes:
  data:
    name: my-app-data
```

它也可以与 `external` 属性结合使用：

```groovy
version: "3.9"
volumes:
  data:
    external: true
    name: my-app-data
```

### 4. configs

顶层配置声明定义。配置来源只可以是`file`或`external`。

- `file`

  配置文件路径名。该方式将按文件内容创建配置。

- `external`

  如果设置为`true`，那么该配置必定是已经是在外部创建好了的。如果没有则会出现`config not found`错误

- `name`

  配置对象名称。

- `driver` and `driver_opts`

  自定加密驱动和驱动选项参数（键值对），只有使用`docker stack`才支持。

- `template_driver`

  模版驱动名称，目前只支持golang模版。

```groovy
configs:
  my_first_config:
    file: ./config_data
  my_second_config:
    external: true
```

```groovy
configs:
  my_first_config:
    file: ./config_data
  my_second_config:
    external:
      name: redis_config
```



### 5. secrets

顶层`screts`声明定义，`secrets`来源可以是`file`或`external`。

- `file`

  文件路径名，该方式将使用文件内容创建`secret`。

- `external`

  当`external:true`时表示该`secret`已经在外部创建好了。如果没有则会出现`secret not found`错误。

- `name`

  `secret`对象名称。

- `template_driver`

  模版驱动名称，目前只支持golang模版。

```groovy
secrets:
  my_first_secret:
    file: ./secret_data
  my_second_secret:
    external: true
```

```groovy
secrets:
  my_first_secret:
    file: ./secret_data
  my_second_secret:
    external: true
    name: redis_secret
```

