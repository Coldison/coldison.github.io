---
layout: post
title: Docker的使用示例(一)
tags:
  - Docker
  - 容器
categories:
  - 教程
  - Docker使用
banner_img: >-
  https://by3302files.storage.live.com/y4mVKy0WXkP-rrMiBMo2rXOuUEXsECbgSnp8zCY1KkrOks5-HIrzWjyRq-Jesa0RkdQeyWpJuO3HZ_YQhy71cK8HUm-sc5RtA4ZqtnSC6XjpsbQOpJCI-PLq9d0bP2ZaayzKMq42hSibW1eyv8UYdqsrX-PMZCsqxVmDJ99IZsjbgX7eCytobaK0ffTFJrX-szw?width=3300&height=1118&cropmode=none
index_img: >-
  https://link.jscdn.cn/1drv/aHR0cHM6Ly8xZHJ2Lm1zL3UvcyFBa3A3T0FGTWxmRUtpWmxCb2xyYTY1bUVHZFVYSnc_ZT1tbjFleXg.png
date: 2023-02-14 20:18:37
---


[Docker 官方文档](https://docs.docker.com/reference/)

现在我们可以来实际实践一下docker技术。我这里的需求，在docker容器中运行c，通过host的端口连接指定的IP地址。这是基本的描述，还有如下的细节：

+ 程序运行时需要在root目录下创建一个conf文件，因此需要先尝试docker的terminal。
+ 程序作为模拟节点会从一个txt里面读取名称，希望和暴露的端口一起都作为镜像的参数，并且将参数写入相应的文件。

## 基础镜像CentOS测试

### 将普通用户加入docker用户组

普通用户需要加入docker用户组才能够使用docker。

```sh
# 切换成root用户
su - 
sudo groupadd docker     #添加docker用户组
sudo gpasswd -a $USER docker     #将登陆用户加入到docker用户组中
newgrp docker     #更新用户组
docker ps    #测试docker命令是否可以使用sudo正常使用
```

### dockerbuild镜像

由于docker是一层一层叠加，我们可以先从简单的开始，得到的镜像层可以复用。
简单的dockerfile:

```dockerfile
FROM centos:7
```

简单的docker build命令：

```bash
docker build -t test .
```

``-t``命令是指给镜像加一个tag名，对于镜像的识别除了依靠分配的十六进制的ID，主要依靠tag名，不同的tag在dockerhub中意味着不同的镜像。

我们可以使用``docker image ls``查看创建的镜像目录，可以看到有两个镜像，一个是我们从[docker.io](docker.io)中拉取的centos7镜像，另一个是我们加了tag之后的镜像。

![docker image ls](https://link.jscdn.cn/1drv/aHR0cHM6Ly8xZHJ2Lm1zL3UvcyFBa3A3T0FGTWxmRUtoOUVvTkozUEZhU2NiVm9NN1E_ZT1FNkRCNVg.png)

### 根据镜像创建容器

现在有了镜像我们可以根据镜像创建容器。可以查看官方文档：[docker run reference](https://docs.docker.com/engine/reference/run/)。

基础的``docker run``命令有如下的形式：

```bash
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```

镜像的开发者可以决定镜像的default设置：

+ 前台还是后台运行
+ 容器识别
+ 网络设置
+ 运行时的约束如CPU和内存

``docker run``命令可以重写覆盖镜像的几乎所有缺省设置，这也是为什么这条命令比其他docker命令复杂的原因。

### 容器的命令和交互

在detached的模式下，当``docker run``运行容器的进程结束时，容器也会进入exited的状态。如果再加上``-rm``命令，则``docker run``进程结束或者守护进程结束时就会进入exited状态。对于有些即时的服务，这条命令可以轻松实现服务结束container退出，然后简单销毁就可以了。

![docker run -d](https://link.jscdn.cn/1drv/aHR0cHM6Ly8xZHJ2Lm1zL3UvcyFBa3A3T0FGTWxmRUtoOUV3NUFtYWliZTluUk5zTEE_ZT10WlNuN1M.png)

如果不加``-d``命令，容器就会处于foreground模式，有如下选项：

```yml
-a=[]           : Attach to `STDIN`, `STDOUT` and/or `STDERR`
-t              : Allocate a pseudo-tty
--sig-proxy=true: Proxy all received signals to the process (non-TTY mode only)
-i              : Keep STDIN open even if not attached
```

通常会使用``-it``表示开启一个可交互的tty。

```bash
(base) [coldison@localhost ~]$ docker run -it  test /bin/bash
[root@2bc4bad23cac /]# ls
anaconda-post.log  bin  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@2bc4bad23cac ~]# exit
exit
(base) [coldison@localhost ~]$
```

输入``exit``或者键盘命令``Ctrl+D``就可以退出tty。可以使用``docker attach``命令进入正在执行某个命令的终端，不能在其中操作；多个窗口同时attach到同一个容器时，所有窗口同步显示；当某个窗口因命令阻塞，其他窗口也无法操作。简而言之，attach命令实际上是绑定到``ENTRYPOINT/CMD``进程的输入输出流上，因此通常难以交互。

我们可以使用``docker exec``命令进行替代，参见官方文档[docker exec](https://docs.docker.com/engine/reference/commandline/exec/)。

``docker exec``命令的格式为：

```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

实际上也就是``docker run``命令省去创建CONTAINER的过程。OPTIONS如下：

|Name, shorthand|Default|Description|
|:----|:----|:----|
|--detach , -d||Detached mode: run command in the background|
|--detach-keys||Override the key sequence for detaching a container|
|--env , -e||Set environment variables|
|--env-file||Read in a file of environment variables|
|--interactive , -i||Keep STDIN open even if not attached|
|--privileged||Give extended privileges to the command|
|--tty , -t||Allocate a pseudo-TTY|
|--user , -u||Username or UID (format: <name|uid>[:<group|gid>])|
|--workdir , -w||Working directory inside the container|

```bash
(base) [coldison@localhost ~]$ docker exec -it 158e7b0112e1 /bin/bash
[root@158e7b0112e1 /]# ls
anaconda-post.log  dev  home  lib64  mnt  proc  run   srv  tmp  var
bin                etc  lib   media  opt  root  sbin  sys  usr
[root@158e7b0112e1 /]# (base) [coldison@localhost ~]$
(base) [coldison@localhost ~]$ docker exec -it 158e7b0112e1 /bin/bash
[root@158e7b0112e1 /]# exit
```

``158e7b0112e1``是``CONTAINER ID``。需要注意的是，一定要加上执行的命令，并且这条命令的进程是容器的主进程之外的一条进程。此时加上``-d``参数就可以完成一些即时的命令。

### docker run reference其余基本设置

在官方文档的[docker run reference](https://docs.docker.com/engine/reference/run/)中，与detached&foreground平级的还有其他一些基本的概念与设置。总的来说，``docker run``试图使用一条命令从一个已经配置完成的镜像运行容器并且完成所有的设定。

这部分内容与我们要做的事情暂时没有很大的关联，但是过一遍对于我们理解和使用docker非常有帮助，让我们知道docker的作用域，哪些可以设置。

#### Container Identification

操作者可以用如下三种方式识别容器：

|Identifier|type|Example value|
|:-|:-|:-|
|UUID long identifier||"f78375b1c487e03c9438c729345e54db9d20cfa2ac1fc3494b6eb60872e74778"|
|UUID short identifier||"f78375b1c487"|
|Name||"evil_ptolemy"

其中如果没有使用``--name``来给容器命名，那么docker守护进程会自动给容器分配一个名字。

也可以将容器的ID输出到文件中：

```yml
--cidfile="": Write the container ID to the file
```

#### Image Identification

image的引用遵循``image[:tag]``的形式，如``docker run ubuntu:14.04``。

采用v2或者更新的镜像格式的镜像还有一个内容可识别标识符（digest），也就是基于镜像的内容生成加密哈希串。为了避免从公共仓库pull下来的有被人篡改，可以在pull镜像的时候使用``image[@digest]``格式，如

```bash
docker run alpine@sha256:9cacb71397b640eca97488cf08582ae4e4068513101088e9f96c9814bfda95e0 date
```

当在pull镜像的时候，会根据镜像的内容计算digest，如果跟我们写入的不相符，就意味着包被人篡改了。

#### PID settings (--pid)

```yml
--pid=""  : Set the PID (Process) Namespace mode for the container,
             'container:<name|id>': joins another container's PID namespace
             'host': use the host's PID namespace inside the container
```

容器的进程默认开启自己的命名空间，但是也可以使用``--pid``加入host或者其余容器的命名空间。如：

```bash
docker run -it --rm --pid=host myhtop
```

加入其它容器的进程空间可以用来为容器debug。如下图，redis这部分我暂时还不太清楚，因此看看就好：
![redis container debug](https://link.jscdn.cn/1drv/aHR0cHM6Ly8xZHJ2Lm1zL3UvcyFBa3A3T0FGTWxmRUtoOUY1eXQzakRWdlFvRXpDWmc_ZT1McHY0WUY.png)

#### UTS settings (--uts)

UTS命名空间用来设置命名空间中可见的运行进程hostname和domain，默认每个容器，包括``--network=host``也有自己的UTS命名空间，当``--uts=host``时容器使用与host一样的UTS命名空间，而此时``--hostname``和``--domainname``设置不再有效。

```yml
--uts=""  : Set the UTS namespace mode for the container,
       'host': use the host's UTS namespace inside the container
```

#### IPC settings (--ipc)

IPC(POSIX/SysV IPC)命名空间提供了命名的共享内存片段、信号和消息队列[(shared memory segments, semaphores, and message queues)](https://docs.docker.com/engine/reference/run/#IPC%20settings:~:text=shared%20memory%20segments%2C%20semaphores%20and%20message%20queues)。共享内存片段用于以内存级别的速度进程间的通信而非通过网络堆栈，经常被用在数据库或者定制的科学计算和金融服务业的高性能应用。如果这些应用被分解成多个容器，那么就需要共享容器的IPC机制。

```yml
--ipc="MODE"  : Set the IPC mode for the container
```

值的设置如下：

|Value|Description|
|:--|:--|
|""|Use daemon's default.|
|"none"|Own private IPC namespace, with /dev/shm not mounted.|
|"private"|Own private IPC namespace.|
|"shareable"|Own private IPC namespace, with a possibility to share it with other containers.|
|"container: <``_name-or-ID_``>"|Join another ("shareable") container's IPC namespace.|
|"host"|Use the host system’s IPC namespace.|

设置``--ipc=shareable``后容器的IPC命名空间可被共享。

#### [Network settings](https://docs.docker.com/engine/reference/run/#network-settings:~:text=for%20other%20containers.-,Network%20settings%F0%9F%94%97,-%2D%2Ddns%3D%5B%5D%20%20%20%20%20%20%20%20%20%20%20%3A%20Set%20custom)

```yml
--dns=[]           : Set custom dns servers for the container
--network="bridge" : Connect a container to a network
                      'bridge': create a network stack on the default Docker bridge
                      'none': no networking
                      'container:<name|id>': reuse another container's network stack
                      'host': use the Docker host network stack
                      '<network-name>|<network-id>': connect to a user-defined network
--network-alias=[] : Add network-scoped alias for the container
--add-host=""      : Add a line to /etc/hosts (host:IP)
--mac-address=""   : Sets the container's Ethernet device's MAC address
--ip=""            : Sets the container's Ethernet device's IPv4 address
--ip6=""           : Sets the container's Ethernet device's IPv6 address
--link-local-ip=[] : Sets one or more container's Ethernet device's link local IPv4/IPv6 addresses
```

容器的network默认开启并且可以进行任何outgoing的连接，可以使用``docker run --network none``关闭outgoing和incoming的网络连接，这样的话容器只能通过文件或者``STDIN``和``STDOUT``进行IO。**开放端口和连接其他的容器**只能在bridge模式下进行，也就是连接到docker本身的网络驱动，一般也更偏向这种方式。``--dns``命令也可以改写DNS。默认情况下MAC地址是根据分配给容器的IP生成，也可以通过``--mac-address``来改写，docker不会确认MAC地址是否是**唯一**的。

具体设置：

|Network|Description|
|:--|:--|
|none|No networking in the container.|
|bridge(default)|Connect the container to the bridge via veth interfaces.|
|host|Use the host's network stack inside the container.|
|container:<name|id>|Use the network stack of another container, specified via its name or id.|
|NETWORK|Connects the container to a user created network (using docker network create command)|

不同的模式还有一些细节，这里不再赘述。

#### Restart policies (--restart)

``--restart``命令用于指定容器在exit之后的重启策略，具体策略如下：

|Policy|Result|
|:--|:--|
|no|Do not automatically restart the container when it exits. This is the default.|
|on-failure[:max-retries]|Restart only if the container exits with a non-zero exit status. Optionally, limit the number of restart retries the Docker daemon attempts.|
|always|Always restart the container regardless of the exit status. When you specify always, the Docker daemon will try to restart the container indefinitely. The container will also always start on daemon startup, regardless of the current state of the container.|
|unless-stopped|Always restart the container regardless of the exit status, including on daemon startup, except if the container was put into a stopped state before the Docker daemon was stopped.|

在重启时，为了防止服务器过载，daemon会采取延时启动的方式，从100ms开始，不断double知道达到``on-failure``限制或者最大延时1min，或者``docker stop``或者``docker rm -f``。

如果一个容器成功重启（也就是started并且成功运行至少10s），上述的时延会被重置为100ms。在``on-faliure``的模式下可以设置最大重启次数，默认是无限次。

```bash
docker run --restart=on-failure:10 redis
```

可以使用``docker inspect``来确认一个容器重启了多少次：

```bash
docker inspect -f "{{.RestartCount}}" my-container
```

或者上一次启动时间：

```bash
docker inspect -f "{{.State.StartedAt}}" my-container
```

#### Exit Status

当``docker run``无法运行或者退出的时候，会返回一个错误码：

|错误码|描述|
|:--|:--|
|125|docker守护进程错误，指docker run命令本身有问题|
|126|容器的命令无法唤起|
|127|容器的命令无法被找到|
|other|其余的命令错误码可以指定，如``docker run busybox /bin/sh -c "exit 3``|

#### Clean up (--rm)

默认情形下，容器退出之后，其文件系统仍旧保留，这方便我们debug，但是如果是运行短时的foreground进程，那么文件就会非常容易堆积。``--rm``标识会使docker自动清理容器并且在容器退出时移除文件系统。``--rm``会移除和容器相关的**匿名空间**，与``docker rm -v my-container``命令类似 。

```bash
docker run --rm -v /foo -v awesome:/bar busybox top
```

上述的命令中，``/foo``文件夹会被移除但是``/bar``会被保留。

#### Security configuration

|Option|Description|
|:--|:--|
|--security-opt="label=user:USER"|Set the label user for the container|
|--security-opt="label=role:ROLE"|Set the label role for the container|
|--security-opt="label=type:TYPE"|Set the label type for the container|
|--security-opt="label=level:LEVEL"|Set the label level for the container|
|--security-opt="label=disable"|Turn off label confinement for the container|
|--security-opt="apparmor=PROFILE"|Set the apparmor profile to be applied to the container|
|--security-opt="no-new-privileges:true"|Disable container processes from gaining new privileges|
|--security-opt="seccomp=unconfined"|Turn off seccomp confinement for the container|
|--security-opt="seccomp=profile.json"|White-listed syscalls seccomp Json file to be used as a seccomp filter|

这里的安全策略，也就是容器允许使用的权限，需要自行定义。

```bash
docker run --security-opt label=disable -it fedora bash
```

容器disbale安全选项。

```bash
docker run --security-opt  no-new-priviledges -it centos bash
```

阻止容器获得更多权限，这种情形下``su``或者``sudo``不再有用。

#### Specify an init process

``--init``可以指定容器中运行的PID为1的进程，默认是``docker-init``。

#### Specify custom cgroups/自定义cgroups

``--cgroup-parent``参数可以给容器指定一个特定的cgroup，然后可以自定义cgroup可以使用的资源。

#### 资源约束

1. [运行时资源约束/Runtime constraints on resources](https://docs.docker.com/engine/reference/run/#network-settings:~:text=common%20parent%20group.-,Runtime%20constraints%20on%20resources,-%F0%9F%94%97)，如内存、CPU等等，用于调整容器的表现，由于设置比较多，给一个链接在这里。

2. [用户内存约束](https://docs.docker.com/engine/reference/run/#network-settings:~:text=Runtime%20constraints%20on%20resources%F0%9F%94%97)

3. [核内存约束/Kernel memory constraints](https://docs.docker.com/engine/reference/run/。#network-settings:~:text=Kernel%20memory%20constraints%F0%9F%94%97)

4. [CPU共享约束/CPU share constraints](https://docs.docker.com/engine/reference/run/#network-settings:~:text=CPU%20period%20constraint)，默认状态下，所有的容器有相同比例的CPU时间片，但是我们可以通过设置比例来调整每个容器所使用的的CPU时间。

5. [CPU周期与比例约束/CPU period&quota constraint](https://docs.docker.com/engine/reference/run/#network-settings:~:text=100%25%20of%20CPU2-,CPU%20period%20constraint,-%F0%9F%94%97)，约束容器使用的CPU周期的比例。

6. [CPU核约束/Cpuset constraint](https://docs.docker.com/engine/reference/run/#network-settings:~:text=bandwidth%20limiting.-,Cpuset%20constraint,-%F0%9F%94%97)，选择容器使用的CPU核，同样也能够约束容器使用的内存。

7. [阻塞IO带宽约束/Block IO bandwidth(Blkio) constraint](https://docs.docker.com/engine/reference/run/#network-settings:~:text=bandwidth%20limiting.-,Block%20IO%20bandwidth%20(Blkio)%20constraint,-%F0%9F%94%97)，设置容器的阻塞通信时间权重，可以具体到某一个设备，包括设备的读写和IO。

#### 用户组

docker容器进程默认在特定的用户的补充用户组中运行，可以使用如下命令增加用户组。

```bash
docker run --rm --group-add audio --group-add nogroup --group-add 777 busybox id
```

#### 运行时权限和Linux能力/[Runtime privilege and Linux capabilities](https://docs.docker.com/engine/reference/run/#network-settings:~:text=99(nogroup)%2C777-,Runtime%20privilege%20and%20Linux%20capabilities,-%F0%9F%94%97)

Option|Description
:--|:--
--cap-add|Add Linux capabilities
--cap-drop|Drop Linux capabilities
--privileged|Give extended privileges to this container
--device=[]|Allows you to run devices inside the container without the --privileged flag.

在默认情况下，Docker容器是没有特权的，因此不被允许去接入任何的设备，而``--privileged``则允许容器接入所有设备，``--device``则是指定接入特定设备，可以``read``，``write``，``mknod``。还可以通过``--cap-add``和``--cap-drop``增加更多linux系统级的能力，这里就不单独列出。

#### 日志驱动/Logging drivers (--log-driver)

容器可以选择跟daemon不同的日志驱动。

#### 覆盖Dockerfile镜像默认设置

当基于Dockerfile创建镜像时，开发者会有一系列的默认设置，除了``FROM``，``RUN``，``MAINTAINER``，``ADD``之外的命令都可以在``docker run``中覆盖。我们可以一条一条来看。

1. CMD(默认命令或者选项)

    ```bash
    docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
    ```

    创建IMAGE的开发者会用Dockerfile中的CMD命令提供默认的COMMAND指令。

2. ENTRYPOINT(运行时默认执行命令)
    ENTRYPOINT是容器运行时必须执行的命令，有了ENTRYPOINT之后CMD命令的作用是为ENTRYPOINT传入更多的参数。可以使用``--entrypoint``参数改写镜像的ENTRYPOINT，如：

    ```bash
    docker run -it --entrypoint /bin/bash example/redis
    ```

    使用COMMAND为ENTRYPOINT传入更多的参数：

    ```bash
    docker run -it --entrypoint /bin/bash example/redis -c ls -l
    docker run -it --entrypoint /usr/bin/redis-cli example/redis --help
    ```

    或者传入一个空的ENTRYPOINT：

    ```bash
    docker run -it --entrypoint="" mysql bash
    ```

3. EXPOSE(外部接入的端口)

    ```yml
    --expose=[]: Expose a port or a range of ports inside the container.
                These are additional to those exposed by the `EXPOSE` instruction
    -P         : Publish all exposed ports to the host interfaces
    -p=[]      : Publish a container's port or a range of ports to the host
                format: ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort | containerPort
                Both hostPort and containerPort can be specified as a
                range of ports. When specifying ranges for both, the
                number of container ports in the range must match the
                number of host ports in the range, for example:
                    -p 1234-1236:1234-1236/tcp

                When specifying a range for hostPort only, the
                containerPort must not be a range.  In this case the
                container port is published somewhere within the
                specified hostPort range. (e.g., `-p 1234-1236:1234/tcp`)

                (use 'docker port' to see the actual mapping)

    --link=""  : Add link to another container (<name or id>:alias or <name or id>)
    ```

    暴露的端口会绑定到host的随机端口，可以使用``docker port``查看，``link``命令用于连接其他的容器，从而使用其他容器暴露的端口。

4. ENV(环境变量)

    Docker不会设置windows容器的任何环境变量，以下是Linux容器的设置：

    Variable|Value
    :--|:--
    HOME|Set based on the value of USER
    HOSTNAME|The hostname associated with the container
    PATH|Includes popular directories, such as /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    TERM|xterm if the container is allocated a pseudo-TTY

    ``docker run``可以使用一个或者更多的``-e``来覆盖上述的环境变量。如果没有特别指定一个环境变量的值，那么环境变量将会到运行容器的环境中去寻找。

    ```bash
    export today=Wednesday
    docker run -e "deep=purple" -e today --rm alpine env
    ```

5. HEALTHCHECK(健康检查)

6. TMPFS(挂载tmpfs文件系统)

    ```bash
    docker run -d --tmpfs /run:rw,noexec,nosuid,size=65536k my_image
    ```

7. VOLUME(共享文件系统)
8. USER
9. WORKDIR

## 总结

本文主要使用了简单的dockerfile构建了一个基本的容器，参考官方文档详细解释了``docker run``命令。
