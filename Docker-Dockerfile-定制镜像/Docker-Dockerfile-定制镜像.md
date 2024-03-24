---
title: Docker Dockerfile 定制镜像
url: https://mp.weixin.qq.com/s/P-MpHecGNcA4hhsOmRrZVQ
clipped_at: 2024-03-24 08:47:19
category: default
tags: 
 - mp.weixin.qq.com
---


# Docker Dockerfile 定制镜像

![图片](assets/1711241239-2c917b570b9454a57f1978d8336de94d.png "null")

目录：

1.  1. FROM
    
2.  2. COPY
    
3.  3. ADD
    
4.  4. RUN
    
5.  5. CMD
    
6.  6. ENTRYPOINT
    
7.  7. LABEL
    
8.  8. EXPOSE
    
9.  9. ENV
    
10.  10. VOLUME
    
11.  11. USER
    
12.  12. WORKDIR
    
13.  13. ARG
    
14.  14. ONBUILD
    
15.  15. STOPSIGNAL
    
16.  16. SHELL
    

## 1\. FROM

FROM 指令用于指定其后构建新镜像所使用的基础镜像。

```plain
FROM <image>
FROM <image>:<tag>
FROM <image>:<digest>
```

-   • FROM 必须是 Dockerfile 中第一条非注释命令
    
-   • 在一个 Dockerfile 文件中创建多个镜像时，docker17.05 版本以后，FROM 可以多次出现。只需在每个新命令 FROM 之前，记录提交上次的镜像 ID。
    
-   • tag 或 digest 是可选的，如果不使用这两个值时，会使用 latest 版本的基础镜像
    

**多次使用 FROM 的情况时构建与运行分离的场景** 基础镜像`golang:1.10.3`是非常庞大的，因为其中包含了所有的 Go 语言编译工具和库，而运行时候我们仅仅需要编译后的 server 程序 就行了，不需要编译时的编译工具，最后生成的大体积镜像就是一种浪费。

`scratch` 是内置关键词，并不是一个真实存在的镜像。FROM scratch 会使用一个完全干净的文件系统，不包含任何文件。因为 Go 语言编译后不需要运行时，也就不需要安装任何的运行库。FROM scratch 可以使得最后生成的镜像最小化，其中只包含了 server 程序。当然，你也可以 FROM 一个你熟悉的并且空间占用小的镜像，比如：`centos、ubuntu、busybox`等。

```plain
# 编译阶段
FROM golang:1.10.3
COPY server.go /build/
WORKDIR /build
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 GOARM=6 go build -ldflags '-w -s' -o server
# 运行阶段
FROM scratch
# 从编译阶段的中拷贝编译结果到当前镜像中
COPY --from=0 /build/server /
ENTRYPOINT ["/server"]
```

COPY 指令的`--from=0` 参数，从前边的阶段中拷贝文件到当前阶段中，多个 FROM 语句时，0 代表第一个阶段。除了使用数字，我们还可以给阶段命名，比如：

```plain
# 编译阶段 命名为 builder
FROM golang:1.10.3 as builder
# ... 省略
# 运行阶段
FROM scratch
# 从编译阶段的中拷贝编译结果到当前镜像中
COPY --from=builder /build/server /
```

`COPY --from` 不但可以从前置阶段中拷贝，还可以直接从一个已经存在的镜像中拷贝。比如，

```plain
FROM ubuntu:16.04

COPY --from=quay.io/coreos/etcd:v3.3.9 /usr/local/bin/etcd /usr/local/bin/
```

## 2\. COPY

COPY 同样用于复制构建环境中的文件或目录到镜像中。

```plain
COPY <src>... <dest>
COPY ["<src>",... "<dest>"]
COPY --from=builder /build/server /
COPY --from=quay.io/coreos/etcd:v3.3.9 /usr/local/bin/etcd /usr/local/bin/
```

COPY 指令非常类似于 ADD，不同点在于 COPY**只会复制构建目录下的文件，不能使用 URL 也不会进行解压操作。**

## 3\. ADD

ADD 用于复制构建环境中的文件或目录到镜像中。

```plain
ADD <src>... <dest>
ADD ["<src>",... "<dest>"]
```

-   • `<src>`指定源文件位置，`<dest>`来指定目标位置。
    
-   • `<src>`可以是一个构建上下文中的文件或目录，也可以是一个 URL，但不能访问构建上下文之外的文件或目录。
    

ADD 复制一个网络文件：

```plain
ADD http://wordpress.org/test.zip $WORKER_PATH
```

另外，如果使用的是本地归档文件（gzip、bzip2、xz）时，Docker 会自动进行解包操作，类似使用`tar -x`

## 4\. RUN

RUN 用于在镜像容器中执行命令，其有以下两种命令执行方式：

**shell 执行**

在这种方式会在 shell 中执行命令，Linux 下默认使用`/bin/sh -c`，Windows 下使用`cmd /S /C`。

注意：通过 SHELL 命令修改 RUN 所使用的默认 shell

```plain
RUN <command>
```

```plain
RUN /bin/bash -c 'source $HOME/.bashrc; \
echo $HOME'                          #通过 RUN 执行多条命令时，可以通过\换行执行
```

```plain
RUN /bin/bash -c 'source $HOME/.bashrc; echo $HOME'  #同一行中，通过分号分隔命令
```

**exec 执行**

```plain
RUN ["executable", "param1", "param2"]
```

RUN 指令创建的中间镜像会被缓存，并会在下次构建中使用。如果不想使用这些缓存镜像，可以在构建时指定--no-cache 参数，如：`docker build --no-cache`。

## 5\. CMD

CMD 用于指定在容器启动时所要执行的命令。CMD 有以下三种格式：

```plain
CMD ["executable","param1","param2"]
CMD ["param1","param2"]
CMD command param1 param2
```

CMD 不同于 RUN，CMD 用于指定在容器启动时所要执行的命令，而 RUN 用于指定镜像构建时所要执行的命令。

CMD 与 RUN 在功能实现上也有相似之处。如：

```plain
docker run -t -i ghostwritten/web_server /bin/true
```

等价于：

```plain
cmd ["/bin/true"]
```

-   • CMD 在 Dockerfile 文件中仅可指定一次，指定多次时，会覆盖前的指令。
    
-   • docker run 命令也会覆盖 Dockerfile 中 CMD 命令。
    

## 6\. ENTRYPOINT

ENTRYPOINT 用于给容器配置一个可执行程序。也就是说，每次使用镜像创建容器时，通过 ENTRYPOINT 指定的程序都会被设置为默认程序。ENTRYPOINT 有以下两种形式：

```plain
ENTRYPOINT ["executable", "param1", "param2"]
ENTRYPOINT command param1 param2
```

-   • ENTRYPOINT 与 CMD 非常类似，不同的是通过 docker run 执行的命令不会覆盖 ENTRYPOINT
    
-   • docker run 命令中指定的任何参数，都会被当做参数再次传递给 ENTRYPOINT。
    
-   • Dockerfile 中只允许有一个 ENTRYPOINT 命令，多指定时会覆盖前面的设置，而只执行最后的 ENTRYPOINT 指令。
    

docker run 运行容器时指定的参数都会被传递给`ENTRYPOINT`，且会覆盖 CMD 命令指定的参数。如，执行`docker run <image> -d`时，-d 参数将被传递给入口点。

也可以通过`docker run --entrypoint`重写 ENTRYPOINT 入口点。

示例：

```plain
ENTRYPOINT ["/usr/bin/nginx"]
```

Dockerfile

```plain
FROM ubuntu:16.04
RUN apt-get update
RUN apt-get install -y nginx
RUN echo 'Hello World, 我是个容器' > /var/www/html/index.html
EXPOSE 80
ENTRYPOINT ["/usr/sbin/nginx"]
```

构建

```plain
docker build -t="ghostwritten/app" .
```

创建容器

```plain
docker run -i -t  ghostwritten/app -g "daemon off;"
```

`-g "daemon off;"`参数将会被传递给`ENTRYPOINT`，最终在容器中执行的命令为`/usr/sbin/nginx -g "daemon off;"`。

## 7\. LABEL

LABEL 用于为镜像添加元数据，元数以键值对的形式指定：

```plain
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

使用 LABEL 指定元数据时，一条 LABEL 指定可以指定一或多条元数据，指定多条元数据时不同元数据之间通过空格分隔。

```plain
LABEL version="1.0" description="这是一个 Web 服务器" by="ghostwritten"
```

也可以换行

```plain
LABEL multi.label1="value1" \
      multi.label2="value2" \
      other="value3"
```

docker inspect 查看：

```plain
$ docker inspect itbilu/test
"Labels": {
    "version": "1.0",
    "description": "这是一个 Web 服务器",
    "by": "ghostwritten"
},
```

**注意**:Dockerfile 中还有个`MAINTAINER`命令，该命令用于指定镜像作者。但 MAINTAINER 并不推荐使用，更推荐使用 LABEL 来指定镜像作者。如：

```plain
LABEL maintainer="ghostwritten"
```

## 8\. EXPOSE

EXPOSE 用于指定容器在运行时监听的端口：

```plain
EXPOSE <port> [<port>...]
```

EXPOSE 并不会让容器的端口访问到主机。要使其可访问，需要在 docker run 运行容器时通过`-p`来发布这些端口，或通过`-P`参数来发布 EXPOSE 导出的所有端口。

## 9\. ENV

ENV 用于设置环境变量，其有以下两种设置形式：

```plain
ENV <key> <value>
ENV <key>=<value> ...
```

示例：

```plain
ENV $WORKER_PATH /myapp
```

设置后，这个环境变量在 ENV 命令后都可以使用。如：

```plain
WORKERDIR  $WORKER_PATH
```

docker run 可以通过 `-e` 新添环境变量或者覆盖环境变量。

```plain
docker run -tid --name test -e $WORKER_PATH /web -e IP=192.168.1.2 ghostwritten/web:v1.0 
```

-   • `-e $WORKER_PATH /web` 为覆盖`ENV $WORKER_PATH /myapp`
    
-   • `-e IP=192.168.1.2`为新添
    

## 10\. VOLUME

VOLUME 用于创建挂载点，即向基于所构建镜像创始的容器添加卷：

```plain
VOLUME ["/data"]
```

一个卷可以存在于一个或多个容器的指定目录，该目录可以绕过联合文件系统，并具有以下功能：

-   • 卷可以容器间共享和重用
    
-   • 容器并不一定要和其它容器共享卷
    
-   • 修改卷后会立即生效
    
-   • 对卷的修改不会对镜像产生影响
    

VOLUME 创建一个挂载点：

```plain
ENV WORKER_PATH /web
VOLUME [$WORKER_PATH]
```

运行容器时，需`-v`参将能本地目录绑定到容器的卷（挂载点）上，以使容器可以访问宿主机的数据。

```plain
docker run -itd --name web -v ~/data:/web/   ghostwritten/web:v1.0
```

## 11\. USER

USER 用于指定运行镜像所使用的用户 可以使用用户名、UID 或 GID，或是两者的组合。

```plain
USER user
USER user:group
USER uid
USER uid:gid
USER user:gid
USER uid:group
```

`docker run`运行容器时，可以通过`-u`参数来覆盖所指定的用户。

## 12\. WORKDIR

WORKDIR 用于在容器内设置一个工作目录：

```plain
WORKDIR /path/to/workdir
```

通过 WORKDIR 设置工作目录后，Dockerfile 中其后的命令 RUN、CMD、ENTRYPOINT、ADD、COPY 等命令都会在该目录下执行。

```plain
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```

pwd 最终将会在/a/b/c 目录中执行。

docker run 运行容器时，可以通过`-w`参数覆盖构建时所设置的工作目录。

## 13\. ARG

ARG 用于指定传递给构建运行时的变量：

```plain
ARG <name>[=<default value>]
```

示例：

```plain
ARG site
ARG build_user=ghostwritten
```

以上我们指定了 site 和 build\_user 两个变量，其中 build\_user 指定了默认值。在使用 docker build 构建镜像时，可以通过`--build-arg <varname>=<value>`参数来指定或重设置这些变量的值。

```plain
$ docker build --build-arg site=ghostwritten -t ghostwritten/test .
```

## 14\. ONBUILD

ONBUILD 用于设置镜像触发器：

```plain
ONBUILD [INSTRUCTION]
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
```

当所构建的镜像被用做其它镜像的基础镜像，该镜像中的触发器将会被钥触发。

示例：第一个构建镜像的 Dockerfile，文件名为`base.df`

```plain
FROM busybox:latest
WORKDIR /app
RUN touch /app/base-evidence
ONBUILD RUN ls -al /app
```

构建

```plain
docker build -t ghostwritten/onbuild:v1.0 -f base.df .
```

第二个构建镜像的 Dockerfile，文件名`downstream.df`

```plain
FROM ghostwritten/onbuild:v1.0
RUN touch downstream-evidence
RUN ls -al .
```

构建

```plain
docker build -t ghostwritten/onbuild_down:v1.0 -f downstream.df .
```

onbuild 指令在第一次构建时不会执行，在第二次被引用时会首先执行。

## 15\. STOPSIGNAL

STOPSIGNAL 用于设置停止容器所要发送的系统调用信号：

```plain
STOPSIGNAL signal
```

所使用的信号必须是内核系统调用表中的合法的值，如：9、SIGKILL。

## 16\. SHELL

SHELL 用于设置执行命令（shell 式）所使用的的默认 shell 类型：

```plain
SHELL ["executable", "parameters"]
```

SHELL 在 Windows 环境下比较有用，Windows 下通常会有 cmd 和 powershell 两种 shell，可能还会有 sh。这时就可以通过 SHELL 来指定所使用的 shell 类型。

参考：

-   • Docker Dockerfile\[1\]
    
-   • 使用 Dockerfile 定制镜像\[2\]
    
-   • Dockerfile reference\[3\]
    

#### 引用链接

`[1]` Docker Dockerfile: *https://www.runoob.com/docker/docker-dockerfile.html*  
`[2]` 使用 Dockerfile 定制镜像: *https://yeasy.gitbook.io/docker\_practice/image/build*  
`[3]` Dockerfile reference: *https://docs.docker.com/engine/reference/builder/*