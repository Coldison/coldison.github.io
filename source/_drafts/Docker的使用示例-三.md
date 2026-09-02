---
title: Docker的使用示例(三)——dockerfile的使用
tags:
  - docker
  - 容器
categories:
  - 教程
  - Docker使用
banner_img: >-
  https://by3302files.storage.live.com/y4mVKy0WXkP-rrMiBMo2rXOuUEXsECbgSnp8zCY1KkrOks5-HIrzWjyRq-Jesa0RkdQeyWpJuO3HZ_YQhy71cK8HUm-sc5RtA4ZqtnSC6XjpsbQOpJCI-PLq9d0bP2ZaayzKMq42hSibW1eyv8UYdqsrX-PMZCsqxVmDJ99IZsjbgX7eCytobaK0ffTFJrX-szw?width=3300&height=1118&cropmode=none
index_img: >-
  https://link.jscdn.cn/1drv/aHR0cHM6Ly8xZHJ2Lm1zL3UvcyFBa3A3T0FGTWxmRUtpWmxCb2xyYTY1bUVHZFVYSnc_ZT1tbjFleXg.png
---

在[Docker的使用示例(一)](https://coldison.github.io/2023/02/14/Docker%E7%9A%84%E4%BD%BF%E7%94%A8%E7%A4%BA%E4%BE%8B(%E4%B8%80)/)和[Docker的使用示例(二)](https://coldison.github.io/2023/02/14/Docker%E7%9A%84%E4%BD%BF%E7%94%A8%E7%A4%BA%E4%BE%8B(%E4%BA%8C)/)中，我们知道了``docker run``命令及docker的镜像跟容器的管理，理论上我们可以通过命令行来用docker执行任意linux可以执行的程序。

但是我们这里的需求中，不仅要基于一个镜像创建一个容器并执行run语句，还可能对这个容器进行更多的操作，比如增删读写文件，在一个容器中运行多个进程，这是简单的run语句所无法完成的，而直接进入容器命令行操作也比较麻烦。因此，这里引入了dockerfile。

参考：
[Dcokerfile Reference](https://docs.docker.com/engine/reference/builder/)

Docker能够通过读取``Dockerfile``中的指令，来创建一个镜像。``Dockerfile``是一个包含所有镜像创建指令的一个文本文件。

### 形式和断句命令（Parser directives)

``Dockerfile``形式为：

```dockerfile
# Comment
INSTRUCTION arguments
```

+ ``#``为注释，Docker指令通常使用大写，虽然``Dockerfile``的指令其实大小写并不敏感。
+ ``\``可以进行断句，但是不支持在注释中使用。在WIN中，由于``\``通常被用作路径，所以在``Dockerfile``开头使用``# escape=` ``使用`` ` ``代替``\``。
+ 一行语句之前的空格会被忽略。

```dockerfile
# 运行hello world指令
# 注释中\不支持断句
    RUN echo "Hello\
World"
```

在WIN中

```dockerfile
# escape=`

# 运行hello world指令
# 注释中`不支持断句
    RUN echo "Hello`
World"
```

等价于

```dockerfile
# 运行hello world指令
# \不支持断句
RUN echo "Hello World"
```

在创建镜像的时候，Docker按序执行指令，在前面我们提到Docker镜像是一层一层累加的，与指令按序执行吻合，一条语句将创建一层镜像。``Dockerfile``必须用``FROM``指令开头，来表明是继承的什么镜像，当然一些注释和全局参数(ARGS)可以放在``FROM``指令之前。

### 注释

出来上述的替换断句符之外，

### 环境替换

``ENV``命令可以设置环境变量，设置的变量可以在``Dockerfile``中的引用规则类似于``bash``。

+ ``$ variable``可以引用变量。
+ ``{variable: -word}``表明当变量没有被设置时，结果为word。
+ ``{variable: +word}``表明当变量被设置时，结果为word。
+ 转义符``\``可以使``$``失效。

```dockerfile
FROM busybox
ENV FOO=/bar
WORKDIR ${FOO}   # WORKDIR /bar
ADD . $FOO       # ADD . /bar
COPY \$FOO /quux # COPY $FOO /quux
```

环境变量在

+ ADD
+ COPY
+ ENV
+ EXPOSE
+ FROM
+ LABEL
+ STOPSIGNAL
+ USER
+ VOLUME
+ WORKDIR
+ ONBUILD (when combined with one of the supported instructions above)

指令中可以使用。在``ENV``指令中使用时，一旦变量赋值，就不能够再改变。

```dockerfile
ENV abc=hello
ENV abc=bye def=$abc
ENV ghi=$abc

# ghi的值为hello而不是bye
```

### .dockerignore文件

类似于``.gitignore``文件，这里不再赘述。

### FROM

```dockerfile
FROM [--platform=<platform>] <image> [AS <name>]
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]
```

``FROM``命令是``Dockerfile``起始所必须的语句，用于创建基本镜像。

+ 如前所述``ARG``和注释是唯二可以置于``FROM``语句之前的命令。
+ ``FROM``命令可以在一个``Dockerfile``命令中出现多次，用于创建多个镜像，或者用于创建镜像过程中的依赖。 ``FROM``命令会清除之前的指令状态重新开始，所以需要在新的``FROM``命令前提交一下之前创建的镜像。
+ ``AS <name>``是一个可选项，用于指定该创建阶段的名称，可以参考[Multi-stage builds](https://docs.docker.com/build/building/multi-stage/)
+ ``tag``是标签，``digest``为识别符，参见[Container Identification](https://coldison.github.io/2023/02/14/Docker%E7%9A%84%E4%BD%BF%E7%94%A8%E7%A4%BA%E4%BE%8B(%E4%B8%80)/#:~:text=YML-,Image%20Identification,-image%E7%9A%84%E5%BC%95%E7%94%A8)。如果不加，则会使用最新的标签。

```dockerfile
ARG VERSION=latest
FROM busybox:$VERSION
ARG VERSION
RUN echo $VERSION > image_version
```

### RUN

``RUN``有两种形式：

+ ``RUN <command>``，在linux种是bash格式，在windows中是cmd格式。
+ ``RUN  ["executable", "param1", "param2"]``，``exec``格式主要参照``JSON array``，因此**此处的参数必须使用""进行引用**，**WIN**格式的路径必须要使用转义符，如``RUN ["c:\\windows\\system32\\tasklist.exe"]``。

``RUN``会在当前镜像的基础上的新一层执行命令并将结果提交，结果将用于新一层的镜像构建。同样的命令产生的Cache会被重复使用，但是也可以使用``docker build --no-cache``强制重新执行命令。

**Issue 783**： 在使用AUFS文件系统中企图删除文件时，可能会出现783权限问题。

#### ``RUN --mount``

```dockerfile
--mount=[type=<TYPE>][,option=<value>[,option=<value>]...]
```

``RUN --mount``允许你创建一个build程序可以访问的文件系统。可以用于(暂时不能理解)：

+ Create bind mount to the host filesystem or other build stages
+ Access build secrets or ssh-agent sockets
+ Use a persistent package management cache to speed up your build

##### Mount types

|Type|Description|
|:--|:--|
|bind (default)|Bind-mount context directories (read-only).|
|cache|Mount a temporary directory to cache directories for compilers and package managers.|
|secret|Allow the build container to access secure files such as private keys without baking them into the image.|
|ssh|Allow the build container to access SSH keys via SSH agents, with support for passphrases.|

+ ``RUN --mount=type=bind``
  + target: Mount路径
  + source: 根路径，默认为文件系统的root目录
  + from: 镜像名或者创建阶段的名称，用于确定文件系统
  + rw, readwrite: 允许写入，但是写入数据最终会被废弃

+ ``RUN --mount=type=cache``
  + id: Optional ID to identify separate/different caches. Defaults to value of target.
  + target1: Mount path.
  + ro,readonly: Read-only if set.
  + sharing: One of **shared, private, or locked**. Defaults to shared. A shared cache mount can be used concurrently by multiple writers. private creates a new mount if there are multiple writers. locked pauses the second writer until the first one releases the mount.
  + from: Build stage to use as a base of the cache mount. Defaults to empty directory.
  + source: Subpath in the from to mount. Defaults to the root of the from.
  + mode: File mode for new cache directory in octal. Default 0755.
  + uid: User ID for new cache directory. Default 0.
  + gid: Group ID for new cache directory. Default 0.

  build不应该假定cache的内容，因为随时可能被另一位用户更改。例子：

  ```dockerfile
  # syntax=docker/dockerfile:1
  FROM golang
  RUN --mount=type=cache,target=/root/.cache/go-build \
    go build ...
  ```

  ```dockerfile
  # syntax=docker/dockerfile:1
  FROM ubuntu
  RUN rm -f /etc/apt/apt.conf.d/docker-clean; echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' > /etc/apt/apt.conf.d/keep-cache
  RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt update && apt-get --no-install-recommends install -y gcc
  ```

  locked保证在使用cache时编译时不会被其他用户更改。

+ ``RUN --mount=type=tmpfs``: 这种类型可以使用容器中临时文件系统。
  + target1: Mount path.
  + size: Specify an upper limit on the size of the filesystem.

+ ``RUN --mount=type=secret``: 这种类型允许访问一些安全文件而不用把这些文件带到镜像中。
  + id: ID of the secret. Defaults to basename of the target path.
  + target: Mount path. Defaults to /run/secrets/ + id.
  + required: If set to true, the instruction errors out when the secret is unavailable. Defaults to false.
  + mode: File mode for secret file in octal. Default 0400.
  + uid: User ID for secret file. Default 0.
  + gid:Group ID for secret file. Default 0.

  ```dockerfile
  # syntax=docker/dockerfile:1
  FROM python:3
  RUN pip install awscli
  # access to S3
  RUN --mount=type=secret,id=aws,target=/root/.aws/credentials \
    aws s3 cp s3://... ...
  ```

  ```dockerfile
  docker buildx build --secret id=aws,src=$HOME/.aws/credentials .
  ```

+ ``RUN --mount=type=ssh``: 允许创建的阶段访问SSH，并支持passphrases
  + id: ID of SSH agent socket or key. Defaults to “default”.
  + target: SSH agent socket path. Defaults to ``/run/buildkit/ssh_agent.${N}``.
  + required: If set to true, the instruction errors out when the key is unavailable. Defaults to false.
  + mode: File mode for socket in octal. Default 0600.
  + uid: User ID for socket. Default 0.
  + gid: Group ID for socket. Default 0.

  ```dockerfile
  # syntax=docker/dockerfile:1
  FROM alpine
  # access to gitlab
  RUN apk add --no-cache openssh-client
  RUN mkdir -p -m 0700 ~/.ssh && ssh-keyscan gitlab.com >> ~/.ssh/known_hosts
  RUN --mount=type=ssh \
    ssh -q -T git@gitlab.com 2>&1 | tee /hello
  # "Welcome to GitLab, @GITLAB_USERNAME_ASSOCIATED_WITH_SSHKEY" should be printed here
  # with the type of build progress is defined as `plain`.
  ```

  ```bash
  $ eval $(ssh-agent)
  $ ssh-add ~/.ssh/id_rsa
    (Input your passphrase here)
  $ docker buildx build --ssh default=$SSH_AUTH_SOCK .
  ```

#### ``RUN --network``

```dockerfile
RUN --network=<TYPE>
```

+ default: 使用build默认的网络。
+ none: 该命令执行不接入网络。
+ host: 使用host的网络环境，类似于``docker build --network-host``但是仅限于一条语句；使用host网络需要在buildkitd daemon中开启。

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.6
ADD mypackage.tgz wheels/
RUN --network=none pip install --find-links wheels mypackage
```

### CMD

``CMD``命令有三种形式：

+ ``CMD ["executable","param1","param2"]``，exec形式，``JSON`` array形式。
+ ``CMD ["param1","param2"]``，默认参数在ENTRYPOINT中。
+ ``CMD command param1 param2``，shell形式。

**不同于shell形式，exec并不唤起一个shell，除非指定执行shell解释器，如用``CMD ["sh", "-c", "echo $HOME"]``替换``CMD [ "echo", "$HOME" ]``。

和``RUN``命令不同的是，``CMD``程序在运行镜像的时候执行。

### LABEL

```dockerfile
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

```dockerfile
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```

```dockerfile
LABEL multi.label1="value1" multi.label2="value2" other="value3"
```

```dockerfile
LABEL multi.label1="value1" \
      multi.label2="value2" \
      other="value3"
```

同样也要使用双引号代替单引号。

可以使用``docker image inspect``命令查看标签，可以加上``--format``只显示关键字。

```bash
docker image inspect --format='' myimage
```

```bash
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

### EXPOSE

```dockerfile
EXPOSE <port> [<port>/<protocol>...]
```

``EXPOSE``命令会告诉Docker容器运行时会监听特定的网络端口，可以指定要使用UDP协议还是TCP协议，默认TCP。

```dockerfile
EXPOSE 80/tcp
EXPOSE 80/udp
```

``EXPOSE``命令并不会发布一个端口，而是作为镜像创建者和容器运行者之间的文档，可以使用``-p``参数在``docker run``命令中发布或者映射端口，或者使用``-P``关键字来发布容器所有被暴露的端口并且映射到更高级的端口上。

### ENV

```dockerfile
ENV <key>=<value> ...
```

前面已经说过怎么配置环境变量，可以使用``docker inspect``查看镜像的环境变量，也可以使用``docker run --env ENV <key>=<value> ...``覆盖，环境变量直到容器运行都有效，在多阶段构建(multi-staged build)中也保持不变。如果环境变量只在build过程中使用，则不必使用``ENV``命令，直接在对应的命令中使用:

```dockerfile
RUN DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -y ...
```

或者使用``ARG``，同样只在build阶段有效:

```dockerfile
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y ...
```

环境设置也可以省略等号:

```dockerfile
ENV MY_VAR my-value
```

如此的话就不支持多重赋值。

### ADD

``ADD``命令从``<src>``复制新文件、目录或者远程文件的URL，并把文件加入到本镜像的文件系统的``<dest>``路径下。

有两种形式：

```dockerfile
ADD [--chown=<user>:<group>] [--checksum=<checksum>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]
```

其中``--chown``只在Linux系统中有效。

``<src>``为到build当前目录下的相对目录，也支持文件路径的匹配规则，如

```dockerfile
# 添加所有前缀hom的文件
ADD hom* /mydir/

# 用任何单一字符替换？，添加匹配的文件
ADD hom?.txt /mydir/
```

``<dest>``是一个绝对路径或者相对于``WORKDIR``的路径。加上``/``就是绝对路径，反之则是相对路径。

```dockerfile
ADD test.txt relativeDir/
ADD test.txt /absoluteDir/
```

如果文件名里有特殊字符，遵循golang规则：

```dockerfile
# 添加arr[0].txt
ADD arr[[]0].txt /mydir/
```

所有添加的文件，默认UID和GID都是0，可以使用``--chown``来给出特定的用户名或者组名。

```dockerfile
ADD --chown=55:mygroup files* /somedir/
ADD --chown=bin files* /somedir/
ADD --chown=1 files* /somedir/
ADD --chown=10:11 files* /somedir/
```

如果一个容器的文件系统没有``/etc/passwd``或者``/etc/group``文件，或者使用的用户名不在这个文件之中，则``ADD``操作会失败，使用数值的UID和GID不存在这个问题。

如果不使用``dockerfile``文件而是标准输入流来进行创建过程，如``docker build - < somefile``，这里没有上下文，所以默认为dockerfile所在的目录内。如果``<src>``是一个URL，那么``<dest>``必须有600权限，如果URL需要授权或者加密，则只能使用``RUN wget``，``RUN curl``等容器内部的命令。如果``<dest>``以反斜杠结尾，则会复制到文件夹下，如果不以反斜杠结尾，则会复制到文件上。如果``<src>``是一个目录，则内容及文件系统都会被复制进去。