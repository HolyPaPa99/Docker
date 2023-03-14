# Docker修改容器系统时区

下面介绍两种修改容器系统时区的方法有：

## 1. 在Dockerfile里修改
```
#设置镜像的时区（不同操作系统命令不一样， 核心思想是在创建镜像时执行系统时区设定命令）
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
echo 'Asia/Shanghai' > /etc/timezone
```
## 2. 启动容器时修改
```
docker run -d \
-v ~/jenkins_home:/var/jenkins_home \
-e TZ=Asia/Shanghai \
-p 7070:8080 \
-p 5000:5000 \
--name jenkins \
jenkins
```