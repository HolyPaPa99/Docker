# Dockerfile语法介绍

## 一. Dockerfile的一般结构
```
FROM microsoft/windowsservercore

# Executed as cmd /S /C echo default
RUN echo default

# Executed as cmd /S /C powershell -command Write-Host default
RUN powershell -command Write-Host default

# Executed as powershell -command Write-Host hello
SHELL ["powershell", "-command"]
RUN Write-Host hello

# Executed as cmd /S /C echo hello
SHELL ["cmd", "/S", "/C"]
RUN echo hello
```

## 二. Dockerfile语法符号

### 1. FROM
语法格式：
```
FROM [--platform=<platform>] <image> [AS <name>]
```
```
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
```
```
FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]
```
参数说明：
- --platform 
可选， --platform 标志可用于在 FROM 引用多平台镜像的情况下指定镜像的平台。 例如，linux/amd64、linux/arm64 或 windows/amd64。 默认情况下，使用构建请求的目标平台。
- image 
基础镜像名称
- tag 和 digest
基础镜像标签或摘要值是可选的。 如果您省略其中任何一个，构建器默认采用最新标签。 如果构建器找不到标签值，它会返回一个错误。
- AS name
可选，通过将 "AS 名称"添加到 FROM 指令来为新构建指定名称。 该名称可用于后续的 FROM 和 COPY --from=<name> 指令以引用在此阶段构建的映像。

FROM 指令初始化一个新的构建并为后续指令设置基础镜像。 因此，有效的 Dockerfile 必须以 FROM 指令开头。 镜像可以是任何有效的镜像——公共仓库中的镜像或是你自己已经创建好的其他镜像。

### 2. RUN
语法格式：
```
RUN <command>
```
```
RUN ["executable", "param1", "param2"]
```
在创建的镜像内执行脚本命令，如创建用户，修改系统时区等等。
例子：
```
##修改系统时区
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
echo $TZ > /etc/timezone
```
### 3. CMD

语法格式：
```
##执行格式, 推荐使用
CMD ["executable","param1","param2"] 
```
```
##作为ENTRYPOINT的默认输入参数
CMD ["param1","param2"] 
```
```
##shell命令格式
CMD command param1 param2 
```
一个 Dockerfile 中只能有一个 CMD 指令。 如果你列出了多个 CMD，那么只有最后一个 CMD 会生效。CMD 的主要目的是为正在执行的容器提供默认值。 这些默认值可以包含可执行命令，也可以省略可执行命令，如果省略可执行命令则必须指定 ENTRYPOINT 指令。另外，如果 CMD 用于为 ENTRYPOINT 指令提供默认参数，则应使用 JSON 数组格式指定 CMD 和 ENTRYPOINT 指令。

### 4. LABEL
语法格式：
```
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```
LABEL 指令用于向镜像添加元数据。 如果要在 LABEL 值中包含空格，请像在命令行解析中一样使用引号和反斜杠。

例子：
```
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```
查看一个镜像的元数据可以使用docker inspect命令：
```
docker image inspect --format='' myimage
```
结果：
```
{
  "com.example.vendor": "ACME Incorporated",
  "com.example.label-with-value": "foo",
  "version": "1.0",
  "description": "This text illustrates that label-values can span multiple lines.",
  "multi.label1": "value1",
  "multi.label2": "value2",
  "other": "value3"
}
```
### 5.  MAINTAINER（已经弃用，被LABEL所取代）
语法格式：
```
MAINTAINER <name>
```

### 6. EXPOSE

语法格式：
```
EXPOSE <port> [<port>/<protocol>...]
```
参数说明：
- port 
监听端口
- protocol 
指定协议

EXPOSE 指令通知 Docker 容器在运行时监听指定的网络端口。 可以指定端口是监听TCP还是UDP，如果不指定协议，默认为TCP。EXPOSE 指令实际上并不发布端口。 它充当构建镜像的人和运行容器的人之间的一种文档描述，关于打算发布哪些端口。 要在运行容器时实际发布端口，请在 docker run 上使用 -p 标志发布和映射一个或多个端口，或使用 -P 标志发布所有暴露的端口并将它们映射到高阶端口。

### 7. ENV
语法格式：
```
ENV <key>=<value> ...
```
设置构建过程中以及容器运行时的环境变量，并且一次可以同时设置多个变量。可以在docker run命令中通过-e参数修改。

如果该环境变量只是在构建过程中使用而不需要在容器运行时也生效那么可以考虑在执行命令时单独设置：
```
RUN DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -y ...
```

### 8. ADD
语法格式：
```
ADD [--chown=<user>:<group>] <src>... <dest>
```
```
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]
```
参数说明：
- --chown 
指定给定的用户名、组名或 UID/GID 组合来请求添加内容的特定所有权。
该功能仅在用于构建 Linux 容器的 Dockerfile 上受支持，不适用于 Windows 容器。 由于用户和组所有权概念不会在 Linux 和 Windows 之间转换，因此使用 /etc/passwd 和 /etc/group 将用户和组名称转换为 ID 限制了此功能仅适用于基于 Linux 操作系统的容器。
- src
源文件路径，可以是文件路径的正则表达式
- dest
绝对路径或相对于 WORKDIR 的路径，源文件将在目标容器内复制到该路径中。其中以“/”开头表示绝对路径。

ADD 指令用于将源文件路径的文件、目录或远程文件 URL复制到镜像文件系统的目标路径。如果路径中包含空格需使用引号。

### 9. COPY
语法格式：
```
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]
```
***和ADD指令类似，区别在于COPY指令的源文件不能使用URL。***

### 10. ENTRYPOINT
语法格式：
```
#命令执行格式（推荐）
ENTRYPOINT ["executable", "param1", "param2"]
```
```
#shell格式
ENTRYPOINT command param1 param2
```
将镜像容器配置成可执行的。docker run 的命令行参数将附加在***命令执行格式***的 ENTRYPOINT 中的所有元素之后，并将覆盖CMD命令指定的所有元素。也就是说可以通过docker run传参，但也可以使用 docker run --entrypoint 覆盖 ENTRYPOINT 指令。

shell 格式的ENTRYPOINT阻止使用任何 CMD 或docker run命令行参数，但缺点是 ENTRYPOINT 将作为 /bin/sh -c 的子命令启动，它不传递信号。 这意味着可执行文件不会是容器的 PID 1 - 并且不会接收 Unix 信号 - 因此您的可执行文件不会从 docker stop收到 SIGTERM。

每个Dockerfile 中的只有最后一条 ENTRYPOINT 指令会起作用。

### 11. VOLUME
语法格式：
```
VOLUME ["/data"]
```

VOLUME 指令创建一个具有指定名称的挂载点，并将其标记为保存到宿主机或其他容器的外部挂载卷。 该值可以是 JSON 数组例如：***VOLUME ["/var/log/"]***或是带有多个参数的纯字符串，例如：***VOLUME /var/log*** 或 ***VOLUME /var/log /var/db***。 

docker run 命令创建新挂载卷并且将镜像制定路径已有的任何数据初始化到新创建的卷中。

注意：
- 在基于window的容器中，不能使用不存在或者空挂在路径，也不能挂载C：盘。
- 从 Dockerfile 中更改卷：***如果任何构建步骤在声明卷后更改了卷中的数据，则这些更改将被丢弃***。
- JSON 格式：列表会被解析为 JSON 数组，***必须用双引号 (") 而不是单引号 (')***。
- ***宿主机目录会在容器运行时声明***：宿主机目录（挂载点）本质上是依赖于宿主机的文件系统。 这是为了保持镜像的可移植性，因为不能保证给定的宿主机目录在所有宿主机上都可用。 因此，无法从 Dockerfile 中挂载宿主机目录。 VOLUME 指令不支持指定主机目录参数，必须在创建或运行容器时指定宿主机挂载点。

### 12. USER
语法格式：
```
USER <user>[:<group>]
```
```
USER <UID>[:<GID>]
```
USER 指令设置镜像运行时所使用的用户名（或 UID）和可选的用户组（或 GID），以及在 Dockerfile 中USER命令之后的任何 RUN、CMD 和 ENTRYPOINT 指令都将以USER指定的用户执行。

注意：
***为用户指定组时，用户将仅具有指定的组成员资格。 任何其他配置的组成员资格都将被忽略。当用户没有设定主用户组时，镜像（或下一个指令）将以root组运行***。

在 Windows 上，如果用户不是内置帐户，则必须先创建该用户。 

### 13. WORKDIR
语法格式：
```
WORKDIR /path/to/workdir
```
WORKDIR 指令为 Dockerfile 中之后的任何 RUN、CMD、ENTRYPOINT、COPY 和 ADD 指令设置工作目录。 如果 WORKDIR 不存在，即使它没有在任何后续 Dockerfile 指令中使用，它也会被创建。

WORKDIR 指令可以在 Dockerfile 中多次使用。 如果提供的是相对路径，它将相对于前一个 WORKDIR 指令的路径。
```
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```
此 Dockerfile 中的最终 pwd 命令的输出将是 /a/b/c。

### 14. ARG
语法格式：
```
ARG <name>[=<default value>]
```
ARG 指令定义了一个构建变量，用户可以在构建时使用 --build-arg <varname>=<value> 标志在执行docker build 命令时将其传递给构建器。 如果用户指定了未在 Dockerfile 中定义的构建参数，则构建会输出警告。ARG 不会像 ENV 那样将变量持久化到构建的映像中。

***不建议使用构建时变量来传递敏感信息，如 github 密钥、用户凭据等。使用 docker history 命令，用户可以看到任何镜像构建时变量值。***

Docker 有一组预定义的 ARG 变量，您无需在 Dockerfile 中使用相应的 ARG 指令即可使用这些变量。
- ***HTTP_PROXY***
- ***http_proxy***
- ***HTTPS_PROXY***
- ***https_proxy***
- ***FTP_PROXY***
- ***ftp_proxy***
- ***NO_PROXY***
- ***no_proxy***

### 15. ONBUILD
语法格式：
```
ONBUILD <INSTRUCTION>
```
ONBUILD 指令将一个指令作为触发器指令添加到镜像中，以便稍后该镜像作为另一个镜像构建的基础镜像时执行。触发器指令将在下游构建的上下文中执行，就好像它是在下游 Dockerfile 中的 FROM 指令之后立即插入的一样。

任何构建指令都可以注册为触发器。

### 16. STOPSIGNAL
语法格式：
```
STOPSIGNAL signal
```
STOPSIGNAL 指令设置系统调用信号将发送到给容器令其退出。 该信号可以是与内核系统调用表中的位置匹配的有效的无符号数，例如 9，或格式为 SIGNAME 的信号名称，例如 SIGKILL。

### 17. HEALTHCHECK
语法格式：
```
#check container health by running a command inside the container
HEALTHCHECK [OPTIONS] CMD command
```
```
#disable any healthcheck inherited from the base image
HEALTHCHECK NONE
```
***OPTIONS参数：***
- --interval=DURATION (default: 30s)
每隔多久执行一次CMD命令
- --timeout=DURATION (default: 30s)
CMD执行超时时间
- --start-period=DURATION (default: 0s)
容器启动时间，在这个时间内健康检查失败将不计算在最大重试次数内，如果在这期间健康检查成功，则认为容器已启动完成。
- --retries=N (default: 3)
CMD指令执行失败重试次数

***CMD命令执行返回结果含义：***
- 0 成功
- 1 失败/不健康
- 2 备用值（一般不使用）

HEALTHCHECK 指令告诉 Docker 如何测试容器以检查它是否仍在工作。 这可以检测诸如 Web 服务器陷入无限循环并且无法处理新连接的情况，即使服务器进程仍在运行。

当容器指定了健康检查时，除了正常状态外，它还有一个健康状态。 这种状态初始值是启动中***starting***。 每当健康检查通过时，它就会变为健康***healthy***（无论它之前处于什么状态）。 在连续失败一定次数后，它变为不健康***unhealthy***。

例如，每五分钟左右检查一次应用服务器是否能够在三秒钟内为网站的主页提供服务：
```
HEALTHCHECK --interval=5m --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```

### 18. SHELL
语法格式：
```
SHELL ["executable", "parameters"]
```

SHELL 指令允许覆盖用于命令的默认 shell形式。 Linux 上的默认 shell 是 ["/bin/sh", "-c"]，Windows 上是 ["cmd", "/S", "/C"]。 SHELL 指令必须以 JSON 格式写入 Dockerfile。

SHELL 指令可以出现多次。 每条 SHELL 指令都会覆盖所有先前的 SHELL 指令，并影响所有后续指令。

SHELL 指令在 Windows 上特别有用，Windows 有两种常用且截然不同的shell形式：cmd 和 powershell，以及可用的备用 shell，包括 sh。

例如：
```
FROM microsoft/windowsservercore

# Executed as cmd /S /C echo default
RUN echo default

# Executed as cmd /S /C powershell -command Write-Host default
RUN powershell -command Write-Host default

# Executed as powershell -command Write-Host hello
SHELL ["powershell", "-command"]
RUN Write-Host hello

# Executed as cmd /S /C echo hello
SHELL ["cmd", "/S", "/C"]
RUN echo hello
```

***参考docker官网文档：https://docs.docker.com/engine/reference/builder/***