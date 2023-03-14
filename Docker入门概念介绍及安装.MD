
# Docker入门概念介绍及安装

Docker 消除了重复、枯燥的配置任务，并在整个开发生命周期中用于快速、简单和可移植的应用程序开发——桌面和云。 Docker 全面的端到端平台包括 UI、CLI、API 和安全性，它们旨在在整个应用程序交付生命周期中协同工作。

## 一. 基本概念

### 1. 镜像构建
即创建一个镜像，它包含安装运行所需的环境、程序代码等。Dockerfile 用来描述镜像的整个构建过程和构建内容，使得docker可以识别并构建出目标镜像。
-  ***一般公共镜像***
一般常见的公共镜像可以从docker hub仓库中拉取, 如centOS， mysql等等， 这些镜像一般都由专业的团队维护，我们直接使用就可以了。

 下面以从docker hub仓库拉取mysql 8.0.26版本镜像为例子：
```
docker pull mysql:8.0.26
```

- ***自定义镜像***
如果仓库的镜像不能够满足需要自定义一个，那就要自己写Dockerfile，例如我想在jenkin容器里配置安装maven：
```
FROM jenkins/jenkins:lts

LABEL author="小小超<xxxxxxxx@qq.com>"

###拷贝maven安装包###
ADD apache-maven-3.8.2-bin.tar.gz /var/local
ADD settings.xml /var/local/apache-maven-3.8.2/conf/settings.xml

###定义环境变量###
ENV MVN_HOME /var/local/apache-maven-3.8.2
ENV PATH /var/local/apache-maven-3.8.2/bin:$PATH

WORKDIR /var/jenkins_home
```

### 2. 启动容器
容器最终运行起来是通过拉取构建好的镜像，使用一系列运行指令（如端口映射、外部数据挂载、环境变量等）来启动服务的。针对单个容器，这可以通过 docker run 来运行。

下面是启动mysql容器的例子：
```
docker run -d \
-p 3306:3306 \
--name mysql \
--network dnet \
--network-alias mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
-e TZ=Asia/Shanghai \
mysql:8.0.26
```

 参数说明：
***-d： 以后台守护进程运行***
***-p： 端口映射（宿主机端口：容器端口）***
***--name： 容器名称***
***--network： 容器网络类型***
***--network-alias： 容器的网络别名***
***-e：环境变量***

容器启动后就可以通过本地端口3306来访问mysql数据库来来：
```
localhost:3306/
```

### 3. 服务编排
一般项目构建会涉及多个镜像容器同时运行的服务可以通过 docker-compose 来实现，***docker-compose.yml可以轻松的将多个容器作为 service 来运行，并且提供了 服务扩容 的功能。***

说了那么多，相信你已经对docker有了一些基本的了解，下面是在CentOS 7操作系统安装docker的方法和一些常用的docker操作命令。

## 二. CentOS 7安装docker
### 1.CentOS操作系统版本要求
  - CentOS 7 64位：系统内核3.10以上
  - CentOS 6.5 要求为64位、系统内核版本为 2.6.32-431以上

  查看内核版本命令：
  ```
  uname -r
  ```

### 2.安装  
  - 更新yum

```
yum update -y
```
  - 卸载老版本
```groovy
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

  - 设置yum的docker下载仓库
```groovy
//安装yum-utils
sudo yum install -y yum-utils
```

```groovy
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

```groovy
//阿里docker源
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

  - 安装最新版本的docker engine 

```
sudo yum install docker-ce docker-ce-cli containerd.io
```
  - 查看可安装docker版本
```groov
yum list docker-ce --showduplicates | sort -r
```

  - 安装指定版本的docker
```groovy
sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```
- 除了上面方法还可以使用官方提供安装脚本安装

```groovy
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

  - 卸载当前版本
```groovy
sudo yum remove docker-ce docker-ce-cli containerd.io
```

```groovy
 sudo rm -rf /var/lib/docker
 sudo rm -rf /var/lib/containerd
```

  - 启动docker
```
systemctl start docker
```

  - 查看docker版本
```
docker version
```
  - 关掉docker
```
systemctl stop docker
```
  - 重启
```
systemctl restart docker
```
  - 开机自启动
```
systemctl enable docker
```
  - 查看运行中的容器
```
docker ps -a
```

### 3.更改安装目录
centos 7 默认安装路径是 /var/lib/docker 

  - 修改存储目录到 /home/data/docker  

```
systemctl stop docker
mkdir -p /home/data/docker
mv /var/lib/docker/* /home/data/docker
cd /var/lib
rm -rf docker
```
  - 进入/home/data/docker 目录建立软连接
```
ln -s /home/data/docker/ /var/lib/docker
```

### 4.配置国内镜像仓库源

docker默认连接的镜像仓库docker hub是国外的，速度你懂的，可以配置为国内的docker镜像仓库。修改配置方法如下：

```groovy
//默認配置文件/etc/docker/daemon.json，可以通過--config-file指令修改
vim /etc/docker/daemon.json
```
输入以下配置即可：
```groovy
{
"registry-mirrors": [
	"https://ung2thfc.mirror.aliyuncs.com",
	"https://registry.docker-cn.com",
	"http://hub-mirror.c.163.com",
	"https://docker.mirrors.ustc.edu.cn"
]
}
```

配置http私有倉庫：

```groovy
{
  "registry-mirrors": ["https://registry.docker-cn.com"],
  "insecure-registries":["192.168.1.30:9030"]
}
```

重启：

```shell
systemctl daemon-reload
systemctl restart docker
```

检查是否生效：

```shell
docker info
```



### 5.删除容器

```
docker stop jenkins
docker rm jenkins
```

### 6.删除镜像
```
docker rmi jenkins/jenkins:lts
```

### 7.使用root用户进入容器
```
docker exec -u 0 -it jenkins bash
```

### 8. docker网络
***Docker安装后，默认会创建下面三种网络类型：bridge, host, none***
- ***bridge 桥接网络（容器启动默认网络）***
  默认情况下启动的Docker容器，都是使用 bridge，Docker安装时创建的桥接网络，每次Docker容器重启时，会按照顺序获取对应的IP地址，这个就导致重启下，Docker的IP地址就变了
- ***host 主机网络***
  使用 --network=none，docker 容器就不会分配局域网的IP。
- ***none 无指定网络***
  使用 --network=host，此时，Docker 容器的网络会附属在主机上，两者是互通的。
  例如，在容器中运行一个Web服务，监听8080端口，则主机的8080端口就会自动映射到容器中。

***查看docker网络***
```
docker network ls
```
***创建一个桥接网络***
```
docker network create <网络名称>
```
***容器启动使用指定网络***
```
docker run -it --name <容器名> ---network <bridge名称> --network-alias <容器网络别名> <镜像名>
```
注意：docker自动创建的bridge网络不能使用容器网络别名，只有自定义创建的网络才可以。

### 9. 給非root用戶授權

```groovy
//將非root用戶添加到docker組，之後重啟
sudo usermod -a -G docker username
```



## 三. docker镜像库

### 1. docker hub

***https://hub.docker.com/***

免费版智能配置一个私有仓库，sad。。。

### 2. 阿里云镜像服务

如果不想自己配置私有仓库服务器的话，可以考虑，适合个人开发。

### 3. Harbor倉庫

