# Docker网络设置

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
