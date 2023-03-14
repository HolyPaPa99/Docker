

## 一. 简介

Docker Machine 是一个工具，可以让你在虚拟或远程主机上安装 Docker 引擎，然后配置 Docker 客户端， 每个托管主机都是一个 Docker 主机和一个配置好的客户端的组合。使用 docker-machine 命令可以配置和管理大量 Docker 主机。您可以使用 Docker Machine 在本地 Mac 或 Windows 机器、公司网络、数据中心或 Azure、AWS 或 DigitalOcean 等云提供商上创建 Docker 主机。

- 在Mac或Windows上安装和运行Docker

  在 Docker v1.12 之前，没有Docker Desktop，Docker Machine 是在 Mac 或 Windows 上运行 Docker 的唯一方法。

- 配置和管理多个远程Docker主机

- 配置Docker Swarm集群

![Docker Machine](image/machine.jpg)

## 二. 安装

macOS:

```groovy
base=https://github.com/docker/machine/releases/download/v0.16.0 \
  && curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/usr/local/bin/docker-machine \
  && chmod +x /usr/local/bin/docker-machine
```

Linux:

```groovy
base=https://github.com/docker/machine/releases/download/v0.16.0 \
  && curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine \
  && sudo mv /tmp/docker-machine /usr/local/bin/docker-machine \
  && chmod +x /usr/local/bin/docker-machine
```

Windows:

```groovy
base=https://github.com/docker/machine/releases/download/v0.16.0 \
  && mkdir -p "$HOME/bin" \
  && curl -L $base/docker-machine-Windows-x86_64.exe > "$HOME/bin/docker-machine.exe" \
  && chmod +x "$HOME/bin/docker-machine.exe"
```



查看是否安装成功：

```groovy
docker-machine version
```



## 三. docker-machine CLI命令

### 1. active

```groovy
docker-machine acitve
```

查看哪台机器是active的（如果 DOCKER_HOST 环境变量指向它，则该机器被认为是活动的）。

### 2. config

```groovy
docker-machine config [OPTIONS] [arg...]
```

打印指定docker主机的连接配置。

参数说明：

- OPTIONS 可选项
  
  ```groovy
  --swarm 显示 Swarm 配置而不是 Docker 守护进程配置
  ```
- arg 

  docker主机名称

例子：

```groovy
docker-machine config dev
```



### 3. create

```groovy
docker-machine create [OPTIONS] [arg...]
```

创建一个docker主机

参数说明：

- OPTIONS

  ```groovy

  --driver 主机驱动，如virtualbox

  --engine-env

  --engine-insecure-registry

  --engine-install-url

  --engine-label

  --engine-opt

  --engine-registry-mirror

  --engine-storage-driver

  --swarm 配置主机加入swarm集群

  --swarm-addr

  --swarm-discovery

  --swarm-experimental

  --swarm-host

  --swarm-image

  --swarm-join-opt

  --swarm-master

  --swarm-opt

  --swarm-strategy

  --tls-san	支持额外域名的TLS认证

  --url 远程主机URL，当--driver none的时候生效。

  --virtualbox-boot2docker-url

  --virtualbox-cpu-count

  --virtualbox-disk-size 

  --virtualbox-host-dns-resolver

  --virtualbox-hostonly-cidr

  --virtualbox-hostonly-nicpromisc

  --virtualbox-hostonly-nictype

  --virtualbox-hostonly-no-dhcp

  --virtualbox-import-boot2docker-vm

  --virtualbox-nat-nictype

  --virtualbox-no-dns-proxy

  --virtualbox-no-share

  --virtualbox-no-vtx-check

  --virtualbox-share-folder

  --virtualbox-ui-type

  ```

- arg

  docker主机名称

### 4. env

```groovy
docker-machine env [OPTIONS] [arg...]
```

显示指定docker主机客户端连接的环境变量设置。

参数说明：

- OPTIONS

  ```groo
  --swarm	Display the Swarm config instead of the Docker daemon
  --shell 	Force environment to be configured for a specified shell: [fish, cmd, powershell, tcsh, emacs], default is auto-detect
  --unset, -u	Unset variables instead of setting them
  --no-proxy	Add machine IP to NO_PROXY environment variable
  ```

  

- arg

  docker主机名称

### 5. help

```groovy
docker-machine help [arg...]
```

显示命令的帮助信息。

参数说明：

- arg 

  需要查看的命令

### 6. inspect

```groovy
docker-machine inspect [OPTIONS] [arg...]
```

输出指定docker主机的详细信息。

参数说明：

- OPTIONS

  ```groovy
  --format, -f 	Format the output using the given go template.
  ```

  

- arg

  docker主机名称

### 7. ip

```groovy
docker-machine ip [arg...]
```

显示指定docker主机的ip地址。

参数说明：

- arg

  docker主机名称

### 8. kill

```groovy
docker-machine kill [arg...]
```

直接杀死指定的 Docker 主机。

参数说明：

- arg

  指定docker主机名称, 可以是多个主机名称以空格分开

### 9. ls

```groovy
docker-machine ls [OPTIONS] [arg...]
```

列出所有docker主机。

参数说明：

- OPTIONS

  ```groovy
  --quiet, -q					Enable quiet mode
  --filter [--filter option --filter option]	Filter output based on conditions provided
  --timeout, -t "10"				Timeout in seconds, default to 10s
  --format, -f 				Pretty-print machines using a Go template
  ```

  

- arg 

  

### 10. mount

```groovy
docker-machine mount [OPTIONS] [arg...]
```

使用 SSHFS 从计算机装载或卸载目录。

参数说明：

- OPTIONS

  ```groovy
  --unmount, -u	Unmount instead of mount
  ```

  

- arg

  ```groovy
  //参数格式
  [machine:][path] [mountpoint]
  ```

  

### 11. provision

```groovy
docker-machine provision [arg...]
```

重新配置指定docker主机。

参数说明：

- arg

  docker主机名称

### 12. regenerate-certs

```groovy
docker-machine regenerate-certs [OPTIONS] [arg...]
```

为某个docker主机重新生成 TLS 信息。

参数说明：

- OPTIONS

  ```groovy
  --force, -f		Force rebuild and do not prompt
  --client-certs	Also regenerate client certificates and CA.
  ```

  

- arg

  docker主机名称，可以指定多个以空格分开。

### 13. restart

```groovy
docker-machine restart [arg...]
```

重启指定docker主机。

参数说明：

- arg

  docker主机名称，可以指定多个以空格分开。

### 14. rm

```groovy
docker-machine rm [OPTIONS] [arg...]
```

删除指定 Docker 主机，对应的虚拟机也会被删除

参数说明：

- OPTIONS

  ```groovy
  --force, -f	Remove local configuration even if machine cannot be removed, also implies an automatic yes (`-y`)
  -y		Assumes automatic yes to proceed with remove, without prompting further user confirmation
  ```

  

- arg

  docker主机名称，可以指定多个以空格分开。

### 15. scp

```groovy
docker-machine scp [OPTIONS] [arg...]
```

在 Docker 主机之间以及 Docker 主机和本地主机之间通过 scp 远程复制文件。

参数说明：

- OPTIONS

  ```groovy
  --recursive, -r	Copy files recursively (required to copy directories)
  --delta, -d		Reduce amount of data sent over network by sending only the differences (uses rsync)
  --quiet, -q		Disables the progress meter as well as warning and diagnostic messages from ssh
  ```

  

- arg

  ```groov
  //格式
  [[user@]machine:][path] [[user@]machine:][path]
  ```

  

### 16. ssh

```groovy
docker-machine ssh [arg...]
```

通过 SSH 连接到主机上，执行命令。

参数说明：

- arg

  ```groovy
  //参数格式
  [machine-name] [command]
  ```

  

### 17. start

```groovy
docker-machine start [arg...]
```

启动一个指定的 Docker 主机，如果对象是个虚拟机，该虚拟机将被启动。

参数说明：

- arg

  docker主机名称，可以指定多个以空格分开。

### 18. status

```groovy
docker-machine status [arg...]
```

获取指定 Docker 主机的状态(包括：Running、Paused、Saved、Stopped、Stopping、Starting、Error)等。

参数说明：

- arg

  docker主机名称。

### 19. stop

```groovy
docker-machine stop [arg...]
```

停止一个指定的 Docker 主机。

参数说明：

- arg

  docker主机名称，可以指定多个以空格分开。

### 20. upgrade

```groovy
docker-machine upgrade [arg...]
```

将一个指定主机的 Docker 版本更新为最新。

参数说明：

- arg

  docker主机名称，可以指定多个以空格分开。

### 21. url

```groovy
docker-machine url [arg...]
```

获取指定 Docker 主机的监听 URL。

参数说明：

- arg

  docker主机名称

## 四. Machine Dirvers

 当 Docker Machine 在本地虚拟机、远程主机或云服务器上配置docker主机时，必须为这些机器提供者们定义对应的驱动和基本操作系统。 Docker Machine支持超过 10 种不同的驱动和一个通用驱动。对于 VirtualBox、Fusion、Hyper-V 等本地虚拟机提供者，默认的基本操作系统是 Boot2Docker。 对于云提供商，基本操作系统是提供商支持的最新 Ubuntu LTS。

每个驱动程序都有一组特定的选项。 这些选项为机器提供信息，例如连接凭据、端口等。

例如：

```groovy
docker-machine create -d azure --azure-subscription-id="SUB_ID" --azure-subscription-cert="mycert.pem" A-VERY-UNIQUE-NAME
```



### 1. Amazon Web Service

***参考：https://docs.docker.com/machine/drivers/aws/***

### 2. DigitalOcean

***参考：https://docs.docker.com/machine/drivers/digital-ocean/***

### 3. Exoscale

***参考：https://docs.docker.com/machine/drivers/exoscale/***

### 4. Generic

通过SSH在已有的VM或主机上配置和管理docker主机。

在创建时驱动程序会执行以下任务：

- 如果 docker 没有在主机上运行，它会自动安装。
- 它更新主机包（apt-get update、yum update...）。
- 它生成证书来保护 docker 守护进程。
- 如果主机使用 systemd，它会创建 /etc/systemd/system/docker.service.d/10-machine.conf
- docker 守护进程重新启动，因此所有正在运行的容器都停止了。
- 主机名已更新以适合机器名称。

可以使用 --generic-ssh-user 标志指定用于通过 SSH 连接到主机的用户。 此用户需要无密码 sudo 权限。 如果不是这种情况，则需要编辑 sudoers 文件，并使用 NOPASSWD 将用户配置为 sudoer。

| 命令行选项                 | 对应环境变量  | 默认值 | 说明 |
| -------------------------- | --------------------- | ------- | :---------- |
| `--generic-engine-port`    | `GENERIC_ENGINE_PORT` | `2376`  | 用于配置Docker 守护程序的端口（boot2docker不适用于） |
| `--generic-ip-address` | `GENERIC_IP_ADDRESS`  | -       | 主机 IP 地址 |
| `--generic-ssh-key`        | `GENERIC_SSH_KEY`     | -       | SSH 用户密钥的路径。 |
| `--generic-ssh-user`       | `GENERIC_SSH_USER`    | `root`  | SSH用户名 |
| `--generic-ssh-port`       | `GENERIC_SSH_PORT`    | `22`    | SSH端口 |

例子：

```groovy
docker-machine create \
  --driver generic \
  --generic-ip-address=203.0.113.81 \
  --generic-ssh-key ~/.ssh/id_rsa \
  vm
```



### 5. Google Compute Engine

***参考：https://docs.docker.com/machine/drivers/gce/***

### 6. IBM Softlayer

***参考：https://docs.docker.com/machine/drivers/soft-layer/***

### 7. Microsoft Azure

***参考：https://docs.docker.com/machine/drivers/azure/***

### 8. Microsoft Hyper-V

***参考：https://docs.docker.com/machine/drivers/hyper-v/***

### 9. OpenStack

***参考：https://docs.docker.com/machine/drivers/openstack/***

### 10. Oracle VirtualBox

使用 VirtualBox 在本地创建机器。 此驱动程序需要在您的主机上安装 VirtualBox 5+。 使用 VirtualBox 4.3+ 应该可以工作，但会发出警告。 旧版本不起作用。

| 命令行选项                           | 对应的环境变量               | 默认值                  | 说明     |
| :----------------------------------- | :--------------------------------- | :----------------------- | ---- |
| `--virtualbox-boot2docker-url`       | `VIRTUALBOX_BOOT2DOCKER_URL`       | *Latest boot2docker url* | boot2docker 镜像的 URL。 默认为最新的可用版本。 |
| `--virtualbox-cpu-count`             | `VIRTUALBOX_CPU_COUNT`             | `1`                      | 用于创建 VM 的 CPU 数量。 默认为单 CPU。 |
| `--virtualbox-disk-size`             | `VIRTUALBOX_DISK_SIZE`             | `20000`                  | 主机的磁盘大小（以 MB 为单位）。 |
| `--virtualbox-host-dns-resolver`     | `VIRTUALBOX_HOST_DNS_RESOLVER`     | `false`                  | 使用主机 DNS 解析器。 （布尔值，默认为false） |
| `--virtualbox-hostonly-cidr`         | `VIRTUALBOX_HOSTONLY_CIDR`         | `192.168.99.1/24`        | 主机的CIDR适配器。 |
| `--virtualbox-hostonly-nicpromisc`   | `VIRTUALBOX_HOSTONLY_NIC_PROMISC`  | `deny`                   | 主机的网络适配器混合模式。 可能的选项是 deny 、allow-vms、allow-all |
| `--virtualbox-hostonly-nictype`      | `VIRTUALBOX_HOSTONLY_NIC_TYPE`     | `82540EM`                | 主机网络适配器类型。 可能的值为“82540EM”（英特尔 PRO/1000）、“Am79C973”（PCnet-FAST III）和“virtio”半虚拟化网络适配器。 |
| `--virtualbox-hostonly-no-dhcp`      | `VIRTUALBOX_HOSTONLY_NO_DHCP`      | `false`                  | 禁用主机 DHCP 服务 |
| `--virtualbox-import-boot2docker-vm` | `VIRTUALBOX_BOOT2DOCKER_IMPORT_VM` | `boot2docker-vm`         | 要导入的 Boot2Docker VM 的名称。 |
| `--virtualbox-memory` | `VIRTUALBOX_MEMORY_SIZE` | `1024` | 主机的内存大小（以 MB 为单位）。 |
| `--virtualbox-nat-nictype` | `VIRTUALBOX_NAT_NICTYPE` | `82540EM` | 指定 NAT 网络适配器类型。 可能的值为“82540EM”（英特尔 PRO/1000）、“Am79C973”（PCnet-FAST III）和“virtio”半虚拟化网络适配器。 |
| `--virtualbox-no-dns-proxy` | `VIRTUALBOX_NO_DNS_PROXY` | `false` | 禁止将所有 DNS 请求代理到主机（布尔值，默认为 false） |
| `--virtualbox-no-share` | `VIRTUALBOX_NO_SHARE` | `false` | 禁止挂载用户主目录 |
| `--virtualbox-no-vtx-check` | `VIRTUALBOX_NO_VTX_CHECK` | `false` | 在 vm 启动之前禁用检查硬件虚拟化的可用性 |
| `--virtualbox-share-folder` | `VIRTUALBOX_SHARE_FOLDER` | - | 挂载指定的目录而不是默认的用户主目录。 |
| `--virtualbox-ui-type` | `VIRTUALBOX_UI_TYPE` | `headless` | 指定UI类型。 |

例子：

```groovy
docker-machine create --driver=virtualbox vbox-test
```

```groovy
docker-machine create -d virtualbox --virtualbox-import-boot2docker-vm boot2docker-vm b2d
```

```groovy
docker-machine create -d virtualbox --virtualbox-disk-size "100000" large
```



### 11. Rackspace

***参考：https://docs.docker.com/machine/drivers/rackspace/***

### 12. VMWare Fusion

***参考：https://docs.docker.com/machine/drivers/vm-fusion/***

### 13. VMWare vCloud Air

***参考：https://docs.docker.com/machine/drivers/vm-cloud/***

### 14. VMWare vSphere

***参考：https://docs.docker.com/machine/drivers/vsphere/***

## 五. 管理虚拟主机和远程主机

### 1. 创建本地virtualbox虚拟docker主机

首先需要安装virtualbox。

macos下可以直接使用home brew安装：

```groovy
brew install --cask virtualbox
```

也可以在官网下载安装程序：https://www.virtualbox.org/wiki/Downloads

使用docker-machine命令创建virtualbox虚拟docker主机机：

```groovy
docker-machine create \
--driver=virtualbox \
--virtualbox-memory=1536 \
--virtualbox-no-share=true \
machine1
```



### 2. 管理远程主机

使用generic驱动管理远程主机：

```groovy
docker-machine create \
  --driver generic \
  --generic-ip-address=xxx.xxx.xxx.xxx \
  --generic-ssh-user=root \
  --generic-ssh-key ~/.ssh/id_rsa \
  aliyuncs1
```

需要使用ssh-key登录，并且登录用户要有免密码sudo权限。

- 生成rsa密钥对

```groovy
//生成rsa公钥和私钥 默认保存路径～/.ssh
//私钥：～/.ssh/id_rsa 公钥：～/.ssh/id_rsa.pub
ssh-keygen -t rsa
```

- 将公钥上传到远程服务器

```groovy
scp ～/.ssh/id_rsa.pub authorized_keys xxx.xxx.xxx.xxx:/root/.ssh/authorized_keys
```



使用docker-machine来管理还是方便很多的，自动安装docker、配置客户端、生成TLS证书。

```groovy
//查看连接配置，包括使用到的客户端TLS证书配置
docker-machine config aliyuncs
```



## 六. 可视化管理工具Portainer.io

Portainer是一个可视化docker主机管理工具，免费版功能有限。操作很简单应用跑起来进去体验一下。

docker安装portainer：

```groovy
docker run -p 9000:9000 -p 8000:8000 --name portainer \
-v /var/run/docker.sock:/var/run/docker.sock \
--restart always \
-d portainer/portainer
```

访问9000端口，第一次访问需创建admin用户。

![portainer](image/portainer.jpg)



***参考：https://docs.docker.com/machine/***