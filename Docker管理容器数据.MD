## 一. 引述
默认情况下，在容器内创建的所有文件都存储在可写容器层上。 这意味着：
- 当该容器不再存在时，数据不会持久存在，并且如果另一个进程需要数据，则很难将数据从容器中取出。
- 容器的可写层与运行容器的主机紧密耦合。 您无法轻松地将数据移动到其他地方。
- 写入容器的可写层需要存储驱动程序来管理文件系统。 存储驱动程序使用 Linux 内核提供联合文件系统。 与使用直接写入主机文件系统的数据卷相比，这种额外的抽象会降低性能。

Docker 有两个选项让容器在主机中存储文件，以便即使在容器停止后文件也能持久化：
- ***卷***
- ***绑定挂载*** 

如果您在 Linux 上运行 Docker，您还可以使用 ***tmpfs 挂载***。 如果您在 Windows 上运行 Docker，您还可以使用***命名管道***。

不管你使用卷、绑定挂载还是 tmpfs 挂载数据都是存储在宿主机上的，他们之间的差别是存储的位置不一样：

![docker数据存储方式](https://upload-images.jianshu.io/upload_images/26973896-d2f105e66967e8de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240 "docker数据存储方式")

- ***卷***
存储在宿主机的文件系统当中由docker管理（Linux默认卷存储路径：/var/lib/docker/volumes/），非docker进程不应该修改这里的内容。卷是docker数据持久化的最佳选择。
- ***绑定挂载***
绑定挂载可以存储在主机系统的任何位置。 它们甚至可能是重要的系统文件或目录。 Docker 主机或 Docker 容器上的非 Docker 进程可以随时修改它们。
- ***tmpfs***
tmpfs 挂载仅存储在主机系统的内存中，永远不会写入主机系统的文件系统。

绑定挂载和卷都可以使用***-v 或 --volume*** 挂载到容器中，但每个的语法略有不同。 对于 tmpfs 挂载，您可以使用 ***--tmpfs***。 我们建议对容器和服务使用 ***--mount*** ，用于绑定挂载、卷或 tmpfs挂载，因为语法更清晰。

***使用卷存储的优势：***
- 与绑定安装相比，卷更容易备份或迁移。
- 可以使用 Docker CLI 命令或 Docker API 管理卷。
- 卷适用于 Linux 和 Windows 容器。
- 卷可以在多个容器之间更安全地共享。
- 卷驱动程序允许您将卷存储在远程主机或云端上，以加密卷的内容或添加其他功能。
- 新卷的内容可以由容器预先填充。
- Docker Desktop上的卷比 Mac 和 Windows 主机上的绑定挂载具有更高的性能。

## 二. 卷
### 1. 概述
卷由 Docker 创建和管理。您可以使用 ***docker volume create*** 命令显式创建卷，或者 Docker 可以在容器或服务创建期间创建卷。

创建卷时，它存储在 Docker 主机上的目录中。当您将卷挂载到容器中时，此目录就是挂载到容器中的目录。这类似于绑定挂载的工作方式，不同之处在于卷由 Docker 管理并且与主机的核心功能隔离。

一个给定的卷可以同时安装到多个容器中。当没有正在运行的容器正在使用卷时，该卷仍然可供 Docker 使用，并且不会自动删除。您可以使用 ***docker volume prune*** 删除未使用的卷。

挂载卷时，它可能是命名的或匿名的。匿名卷在首次挂载到容器中时没有明确的名称，因此 Docker 为它们提供了一个随机名称，该名称保证在给定的 Docker 主机中是唯一的。除了名称之外，命名卷和匿名卷的行为方式相同。

***卷还支持使用卷驱动程序，这允许您将数据存储在远程主机或云端上等等。

### 2. 应用场景
- ***在多个正在运行的容器之间共享数据。***
- ***将 Docker 主机的目录和文件结构配置与容器运行时分离***
- ***将容器的数据存储在远程主机或云端***
- ***将数据从一台 Docker 主机备份、还原或迁移到另一台***
- 使用 Docker Desktop上的高性能 I/O 。
卷存储在 Linux VM 中而不是主机中，这意味着读取和写入具有更低的延迟和更高的吞吐量。
- 使用Docker Desktop操作主机文件系统。

### 3. 创建和管理卷
 - 创建卷
```
docker volume create my-vol
```
- 查看以创建的卷
```
docker volume ls
```
- 查看一个卷的详细信息
```
docker volume inspect my-vol
```
- 删除卷
删除命名卷：
```
docker volume rm my-vol
```
要自动删除匿名卷，请使用 ***--rm*** 选项。 例如，此命令创建匿名 /foo 卷。 当容器被移除时，Docker 引擎会移除 /foo 卷而不是 awesome 卷。
```
docker run --rm -v /foo -v awesome:/bar busybox top
```
删除所有未使用的卷并释放空间：
```
docker volume prune
```
### 4. 使用卷挂载启动容器

#### 4.1 使用-- mount参数挂载卷启动容器
--mount参数值是多个以***","***间隔的***<key>=<value>***键值对，相对于-v参数--mount参数的语意更加明确和详细， 易于理解， 另外如果需要指定卷驱动程序选项，则必须使用--mount。官方推荐使用--mount参数。

***--mount参数的key有如下几种：***
- ***type***
指定挂载类型， 值可以是volume， bind 或者tmpfs。
- ***source（也可以写成src）***
挂载卷的名称，如果是匿名卷则不需要声明。
- ***destination（也可以写成dst或target）*** 
挂载到容器的指定文件/目录路径。
- ***readonly***
表示这个挂载到容器的内容只读。
- ***volume-driver***
卷驱动
- ***volume-opt***
卷驱动可配置选项，值是键值对key=value。

例子：
```
docker service create \
    --mount 'type=volume,src=<VOLUME-NAME>,dst=<CONTAINER-PATH>,volume-driver=local,volume-opt=type=nfs,volume-opt=device=<nfs-server>:<nfs-path>,"volume-opt=o=addr=<nfs-address>,vers=4,soft,timeo=180,bg,tcp,rw"'
    --name myservice \
    <IMAGE>
```

#### 4.2 使用-v参数挂载卷启动容器
```
-v 卷名称:挂载容器的路径名:可选参数
```
-v 参数的值包含三个内容分别以***":"***分隔：
- 卷名称
- 指定的挂载容器路径名
- 第三个是可选的
ro表示readonlhy只读，

例子：
 ```
docker run -d \
  --name devtest \
  -v myvol2:/app:ro \
  nginx:latest
```

### 5. 在服务编排docker-compose中使用卷挂载
一般docker-compose.yml里使用卷挂载如下例子，第一次启动的时候如果卷会被创建，如果卷是在外部通过docker volume create创建的需要做引用声明：
```
version: "3.9"
services:
  frontend:
    image: node:lts
    volumes:
      - myapp:/home/node/app
volumes:
  myapp:
    #如果卷是在外部通过docker volume create命令创建的需要在此做引用声明
    #external: true
```

### 6. 数据共享

![数据共享](https://upload-images.jianshu.io/upload_images/26973896-159013d32545bafd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在开发应用程序时，有多种方法可以实现数据共享。 可以向应用程序添加逻辑，以将文件存储在 Amazon S3 等云对象存储系统上。 当然也可以使用支持将文件写入外部存储系统（如 NFS 或 Amazon S3）的卷驱动程序。

卷驱动程序允许从应用程序逻辑中抽象出底层存储系统。 例如，如果您的服务使用带有 NFS 驱动程序的卷，您可以更新服务以使用不同的驱动程序，例如将数据存储在云中，而无需更改应用程序逻辑。

下面以***vieux/sshfs***卷驱动插件为例子，假设你有两个运行服务节点，其中一个是docker主机，并且docker主机节点可以通过ssh链接到另外一个节点上。
先从docker公共仓库中拉取***vieux/sshfs***插件:
```
 docker plugin install --grant-all-permissions vieux/sshfs
```
下面是通过***docker volume create***命令方式创建卷，这个例子指定了 SSH 密码，但如果两台主机配置了共享密钥，则可以省略密码，每个卷驱动程序可能有零个或多个可配置选项，每个选项都使用 -o 标志指定：
```
docker volume create --driver vieux/sshfs \
  -o sshcmd=test@node2:/home/test \
  -o password=testpassword \
  sshvolume
```
下面是通过启动容器的方式创建卷的，如果卷驱动程序要求您传递选项，则必须使用 --mount 标志来挂载卷，而不是 -v：
```
docker run -d \
  --name sshfs-container \
  --volume-driver vieux/sshfs \
  --mount src=sshvolume,target=/app,volume-opt=sshcmd=test@node2:/home/test,volume-opt=password=testpassword \
  nginx:latest
```

### 7. 卷数据备份、恢复和迁移

假设我们使用ubuntu操作系统运行数据库容器，并将数据库文件存储在***/dbdata***目录中，下面例子将演示备份***/dbdata***目录的内容并从备份文件中恢复数据。

首先启动一个名字叫***dbstore***的容器：
```
#这里使用了匿名卷
 docker run -v /dbdata --name dbstore ubuntu /bin/bash
```
然后从dbstore 容器的挂载卷启动另一个新容器 ，将本地主机目录绑定挂载到容器 /backup目录，并且通过命令行传递将 /dbdata目录的内容压缩到 /backup 目录中的 backup.tar 文件作为备份：
```
#这里使用匿名的方式启动容器通过命令传递在容器内压缩文件创建备份，完成后自动删除容器
docker run --rm --volumes-from dbstore -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata
```
使用刚刚创建的备份，可以将其恢复到同一个容器或在其他地方创建的另一个容器中。例如，我们要将它恢复到一个叫dbstore2的容器中。

首先创建一个名为 dbstore2 的新容器：
```
 docker run -v /dbdata --name dbstore2 ubuntu /bin/bash
```
然后解压tar备份文件：
```
docker run --rm --volumes-from dbstore2 -v $(pwd):/backup ubuntu bash -c "cd /dbdata && tar xvf /backup/backup.tar --strip 1"
```

## 三. 绑定挂载
自 Docker 早期就可用。 与卷相比，绑定挂载的功能有限。 使用绑定挂载时，主机上的文件或目录会挂载到容器中。 文件或目录由其在主机上的完整路径引用。 如果它尚不存在，则按需创建。 绑定挂载非常高效，但它们***依赖于具有特定目录结构的主机文件系统***。 如果您正在开发新的 Docker 应用程序，请考虑改用命名卷。 您不能使用 Docker CLI 命令直接管理绑定挂载。

使用绑定挂载的一个副作用，无论好坏，是您可以通过容器中运行的进程更改主机文件系统，包括创建、修改或删除重要的系统文件或目录。 这是一种强大的能力，可能会产生安全隐患，包括影响主机系统上的非 Docker 进程。

***应用场景：***
- ***从主机到容器共享配置文件。*** 
这就是 Docker 默认为容器提供 DNS 解析的方式，方法是将 /etc/resolv.conf 从主机挂载到每个容器中。
- 在 Docker 主机开发环境和容器之间共享源代码或构建工件。 
例如，您可以将 Maven 的target/目录挂载到容器中，这样每次在 Docker 主机上构建 Maven 项目时，容器都可以访问重建的工件。
***不过真实生产上并不会这样做***
- 当 Docker 主机的文件或目录结构保证与容器所需的绑定挂载一致时，可以使用绑定挂载这种方式。
***从解耦合提高可移植性上来说这种做法并不推荐***


## 四. tmpfs挂载
tmpfs 挂载不会持久保存在磁盘上，无论是在 Docker 主机上还是在容器内。 在容器的生命周期内，容器可以使用它来存储非持久状态或敏感信息。 例如，在内部，swarm 服务使用 tmpfs 挂载将密钥挂载到服务的容器中。

***应用场景：***
tmpfs 挂载最适用于您不希望数据在主机上或容器内持久化的情况。 这可能是出于安全原因或在您的应用程序需要写入大量非持久状态数据时保护容器的性能。

## 五. 存储驱动

## 六. 第三方卷驱动插件

未完待续～～～～

***参考：https://docs.docker.com/storage/***