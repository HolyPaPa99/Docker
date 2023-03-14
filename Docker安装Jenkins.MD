## 1. Dockerfile
```
FROM openjdk:11

# 安装 curl 和 vim
RUN apt-get update && apt-get install -y git curl && echo "Y" | apt-get install vim && rm -rf /var/lib/apt/lists/*

# build参数
ARG user=jenkins
ARG group=jenkins
ARG uid=1000
ARG gid=1000
ARG http_port=8080
ARG agent_port=50000
# 默认下载最新版本
ARG JENKINS_VERSION=latest
# 默认分配1G内存
ARG JAVA_OPTS="-server -Xms800m -Xmx800m -XX:PermSize=64M -XX:MaxNewSize=256m -XX:MaxPermSize=128m -Djava.awt.headless=true"
# Jenkins配置保留参数
ARG JENKINS_OPTS
# jenkins下载url
ARG JENKINS_URL=https://get.jenkins.io/war-stable/${JENKINS_VERSION}/jenkins.war
ARG JENKINS_SHA_URL=https://get.jenkins.io/war-stable/${JENKINS_VERSION}/jenkins.war.sha256

# Maven版本号
ARG MAVEN_VERSION=3.8.2
ARG MAVEN_URL=https://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz
ARG MAVEN_SHA_URL=https://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz.sha512

#设置镜像的时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
echo 'Asia/Shanghai' > /etc/timezone
# 设置jenkins目录
ENV JENKINS_HOME /var/jenkins_home
# 设置slave agent port
ENV JENKINS_SLAVE_AGENT_PORT ${agent_port}


# 创建Jenkins用户和组
RUN groupadd -g ${gid} ${group} \
    && useradd -d "$JENKINS_HOME" -u ${uid} -g ${gid} -m -s /bin/bash ${user}

# jenkins卷挂载目录
VOLUME /var/jenkins_home

# 创建jenkins下载目录
RUN mkdir -p /usr/share/jenkins
# 下载jenkins并验证sha256
RUN curl -fsSL ${JENKINS_URL} -o /usr/share/jenkins/jenkins.war \
&& curl -fsSL ${JENKINS_SHA_URL} -o /usr/share/jenkins/jenkins.war.sha256 \
&& cd /usr/share/jenkins \
&& cat /usr/share/jenkins/jenkins.war.sha256 | sha256sum -c -

# 下载maven并验证sha512
RUN curl -fsSL ${MAVEN_URL} -o /usr/share/jenkins/apache-maven-${MAVEN_VERSION}-bin.tar.gz \
&& curl -fsSL ${MAVEN_SHA_URL} -o /usr/share/jenkins/apache-maven-${MAVEN_VERSION}-bin.tar.gz.sha512 \
&& cd /usr/share/jenkins \
&& echo " apache-maven-${MAVEN_VERSION}-bin.tar.gz" >> apache-maven-${MAVEN_VERSION}-bin.tar.gz.sha512 \
&& sha512sum -c apache-maven-${MAVEN_VERSION}-bin.tar.gz.sha512 \
&& tar xvf apache-maven-${MAVEN_VERSION}-bin.tar.gz \
&& mv apache-maven-${MAVEN_VERSION} maven

# 如果主机需要替换settings.xml可以通过绑定挂载覆盖
ADD --chown=${user}:${group} settings.xml /usr/share/jenkins/maven/conf/settings.xml
ENV MVN_HOME /usr/share/jenkins/maven
ENV PATH $PATH:$MVN_HOME/bin

# docker host 默认bridge网络下主机局域网ip， 自定义网络可在docker run中设置-e DOCKER_HOST=xxxx 覆盖
# Jenkins使用maven来build镜像的时候，因为容器里没有安装docker所以通过docker主机来build
ENV DOCKER_HOST tcp://172.17.0.1:2375

# jenkins应用服务端口
EXPOSE ${http_port}

# slave agents端口:
EXPOSE ${agent_port}

USER ${user}

WORKDIR /var/jenkins_home

CMD java ${JAVA_OPTS} -jar /usr/share/jenkins/jenkins.war ${JENKINS_OPTS}
#ENTRYPOINT ["java", "-jar", "/usr/share/jenkins/jenkins.war"]

```
## 2. build镜像
```
docker build -t jenkins .
```

## 3. 运行容器
```
docker run -d \
-v jenkins:/var/jenkins_home \
-p 7070:8080 \
-p 5000:5000 \
--name jenkins \
jenkins
```

这里为什么不通过docker主机共享maven， 因为通过绑定挂载的方式共享会涉及写操作的权限问题而且还要在主机配置maven麻烦，所以选择容器内下载安装，一切所需都在镜像里。 如果要在主机共享maven的settings.xml配置文件，把本地仓库路径配置为${user.home}/路径下目录就不涉及权限问题。

另外说明一下，如果在jenkins里使用maven插件dockerfile-maven-plugin构建应用镜像需要在docker主机配置endpoint，并为容器设置DOCKER_HOST环境变量，因为容器内没有安装docker，这个我在Dockerfile也有备注。

***docker主机配置endpoint方法如下：***
- 修改daemon.json配置
```
vi /etc/docker/daemon.json 
```
往daemon.json 添加hosts配置：
```
{
   "hosts": ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"]
}
```
- 重启使配置生效
```
systemctl daemon-reload     
```
```    
systemctl restart docker.service
```
***注意： 现在DockerDesktop版本好像是不支持修改hosts配置的，如果在mac里要使用endpoint可以使用docker-machine配置虚拟机的方式，比较麻烦。***