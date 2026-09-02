---
layout: post
title: Docker的使用示例(二)
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
date: 2023-02-14 20:18:10
---


[Docker 官方文档](https://docs.docker.com/reference/)

现在我们重新回到需求，我们需要在docker中导入一些文件，制作镜像并且希望其顺利运行。在[Docker的使用示例(一)](https://coldison.github.io/2022/09/15/Docker%E7%9A%84%E4%BD%BF%E7%94%A8%E7%A4%BA%E4%BE%8B(%E4%B8%80)/)中，我们可以Build简单的镜像，然后使用``docker run``从这个镜像运行容器，并且为其做一系列的配置。理论上，在镜像已经完成的时候，我们可以直接通过一条``docker run``命令来实现应用的运行。

但是我们也可以只运行简单的容器，用``docker contianer``来管理和监视容器，``docker cp``为容器导入文件，``docker exec``运行指定的命令，``docker commit``等做版本管理，来构建想要的镜像应用。另外还涉及到一些容器或者镜像的管理或者使用上的细节，在本文中我们暂时不会涉及Docker CLI的volume或者dockerfile。

**由于文档阅读和实践和认知的不统一，内容编排上有些可能不太符合认识的过程，敬请谅解。**

## 容器的运行

假定我们已经有了一个容器，这个容器至少是linux runtime，那么我们可以把这个容器当成虚拟机进行操作。

### docker container

``docker container``是一个命令集，帮助我们管理容器。实际上整个[容器的运行](#容器的运行)部分全都是``docker container``命令，只不过因为有些比较常用会把中间的``container``关键字省去，比如``docker run``其实等价于``docker container run``。这里我们介绍一些常用的命令，**如果存在省略关键字而功能同样的命令如``docker run``和``docker container run``，则在一条介绍了用法，另外一条给出超链接**，从而方便日后的归档查询和使用。

#### docker container run

等同于``docker run``命令，参见[docker run](https://coldison.github.io/2022/09/15/Docker%E7%9A%84%E4%BD%BF%E7%94%A8%E7%A4%BA%E4%BE%8B(%E4%B8%80)/#:~:text=%E5%A4%8D%E6%9D%82%E7%9A%84%E5%8E%9F%E5%9B%A0%E3%80%82-,%E5%AE%B9%E5%99%A8%E7%9A%84%E5%91%BD%E4%BB%A4%E5%92%8C%E4%BA%A4%E4%BA%92,-%E5%9C%A8detached%E7%9A%84)。

#### docker container ls

```bash
docker container ls [OPTIONS]
```

选项：

|Name, shorthand|Default|Description|
|:--|:--|:--|
|--all , -a||Show all containers (default shows just running)|
|--filter , -f||Filter output based on conditions provided|
|--format||Pretty-print containers using a Go template|
|--last , -n|-1|Show n last created containers (includes all states)|
|--latest , -l||Show the latest created container (includes all states)|
|--no-trunc||Don't truncate output|
|--quiet , -q||Only display container IDs|
|--size , -s||Display total file sizes|

#### docker container start|restart|pause|stop|kill

docker容器的启停关闭和杀掉容器进程，这里省略部分新特性，如果遇到了可以查一下官方文档。

1. ``docker container start``

    ```bash
    docker container start [OPTIONS] CONTAINER [CONTAINER...]
    ```

    选项：

    |Name, shorthand||Default Description|
    |:--|:--|:--|
    |--attach , -a||Attach STDOUT/STDERR and forward signals|
    |--detach-keys||Override the key sequence for detaching a container|
    |--interactive , -i||Attach container's STDIN|

    **关于标准输入输出流，也就是``docker attach/detach``这些操作，由于我对于这个概念还有一些模糊的地方，所以只了解基本用法，但是应该不影响我们操作。**

2. ``docker container restart``

    ```bash
    docker container restart [OPTIONS] CONTAINER [CONTAINER...]
    ```

    选项：

    |Name, shorthand|Default|Description|
    |:--|:--|:--|
    |--time , -t|10|Seconds to wait for stop before killing the container|

3. ``docker container pause``

    ```bash
    docker pause CONTAINER [CONTAINER...]
    ```

    ``docker container pause``命令会停止容器内执行的所有进程。

4. ``docker container stop``

    ```bash
    docker container stop [OPTIONS] CONTAINER [CONTAINER...]
    ```

    选项：
    |Name, shorthand|Default|Description|
    |:--|:--|:--|
    |--time , -t|10|Seconds to wait for stop before killing it|
5. ``docker container kill``

    ```bash
    docker container kill [OPTIONS] CONTAINER [CONTAINER...]
    ```

    选项：

    |Name, shorthand|Default|Description|
    |:--|:--|:--|
    |--signal , -s|KILL|Signal to send to the container|

#### docker container cp

等同于``docker cp``，参见[docker cp](#docker-cp)。

#### docker container exec

等同于``docker exec``，参见[docker exec](#docker-exec)

#### docker container port

等同于``docker port``，参见[docker port](#docker-port)。

#### docker container commit

``docker container commit``等于``docker commit``。参见[docker commit](#docker-commit)。

### docker cp

```bash
 docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
```

``docker cp``将``SRC_PATH``的内容复制到``DEST_PATH``。可以从本机将文件复制到容器，反之亦然。加上在``SRC_PATH``或者``DEST_PATH``上加上``-``，就可以从``STDIN``中读取tar archive或者将其输入到``STDOUT``。容器既可以是正在运行的或者是已经终止的，PATH既可以是文件夹也可以是文件。

``docker cp``命令假设容器的路径都是相对于容器的根目录。所以加上``/``是可选的项，不加一样从根目录开始解析路径。

``docker cp``命令和Unix``cp -a``命令类似，文件夹的内容如果权限允许的话都可以递归复制。复制后文件夹的权限被设置为user和primary group。例如，如果是容器内根用户创建的属于``UID:GID``的文件，复制到本机就是属于使用``docker cp``的用户。

加入``-a``命令，文件的所属被设为source的所属。命令默认不去复制link的target，``-L``选项可以解析link。在``DEST_PATH``不存在的时候，文件夹会被创建，``SRC_PATH``的内容而非父文件夹被复制进``DEST_PATH``。当``DEST_PATH``存在的时候，末尾不加``/.``则复制父文件夹，加``./``则复制文件夹内容。

``:``是CONTAINER与PATH之间的定界符，也可以用于描述本机文件。

示例：

+ 复制本地文件到容器

    ```bash
    docker cp ./some_file CONTAINER:/work
    ```

+ 复制容器文件夹到本地

    ```bash
    docker cp CONTAINER:/var/logs/ /tmp/app_logs
    ```

+ 复制容器文件到标准输出流

    ```bash
    docker cp CONTAINER:/var/logs/app.log - | tar x -O | grep "ERROR"
    ```

    传输的是一个tar stream所以需要解码。

边界情况：``docker cp``命令无法复制``/proc``，``/sys``，``/dev``，tmpfs和mounts等用户在容器中创建的资源，但是可以通过在``docker exec``运行``tar``命令来进行。下面命令展示如何将``SRC_PATH``的内容绑定到标准输入流再从``DEST_PATH``中解析：

```bash
 docker exec CONTAINER tar Ccf $(dirname SRC_PATH) - $(basename SRC_PATH) | tar Cxf DEST_PATH -
```

```bash
tar Ccf $(dirname SRC_PATH) - $(basename SRC_PATH) | docker exec -i CONTAINER tar Cxf DEST_PATH -
```

这里``DEST_PATH``必须是文件夹。

### docker exec

参见[docker exec](https://coldison.github.io/2022/09/15/Docker%E7%9A%84%E4%BD%BF%E7%94%A8%E7%A4%BA%E4%BE%8B(%E4%B8%80)/#IPC-settings-%E2%80%93ipc:~:text=docker%20exec%E3%80%82-,docker%20exec,-%E5%91%BD%E4%BB%A4%E7%9A%84%E6%A0%BC%E5%BC%8F)。

通常情况下我们运行：

```bash
docker exec -it CONTAINER /bin/bash
```

然后进入容器进行操作就完全够了。

### docker port

```bash
docker port CONTAINER [PRIVATE_PORT[/PROTO]]
```

显示某个容器的端口映射。

![docker port](https://link.jscdn.cn/1drv/aHR0cHM6Ly8xZHJ2Lm1zL3UvcyFBa3A3T0FGTWxmRUtpWmt6d2ViZXhMM0dPVlN3UGc_ZT1DblZqaEM.png)

## Docker管理

对于镜像和容器的版本管理和维护，加载或者导出。

### docker commit

```bash
 docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

通常更建议使用dockerfile来对镜像进行维护，但是也可以通过这种方式在容器中开启一个交互命令进行debug，也可以讲一个正在工作的数据集上传到另外一个服务器上。``docker commit``命令不涉及volume的操作。

OPTIONS:

|Name, shorthand|Default|Description|
|:--|:--|:--|
|--author , -a||Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")|
|--change , -c||Apply Dockerfile instruction to the created image|
|--message , -m||Commit message|
|--pause , -p|true|Pause container during commit|

默认情况下，当commiting镜像时，正在commited的容器和进程会被暂停，以减小数据崩溃的可能性。如果希望取消，可以置``--pause``flag为``False``。

``--change``可以将任意的dockerfile命令应用到images当中。

commit容器和新的configurations:
![commit a container with configurations](https://link.jscdn.cn/1drv/aHR0cHM6Ly8xZHJ2Lm1zL3UvcyFBa3A3T0FGTWxmRUtpWmtNMEt1RmpqaWtrWEthUHc_ZT1Ia1RncXc.png)

commit容器和新的命令：
![commit a container with new CMD and EXPOSE instructions](https://link.jscdn.cn/1drv/aHR0cHM6Ly8xZHJ2Lm1zL3UvcyFBa3A3T0FGTWxmRUtpWmtPblNsdnFHVW51eGtsWUE_ZT16WjFmYVM.png)

### docker save

``commit``是将容器的改变传给镜像，而``save``是将一个或者多个镜像打包成tar包，默认绑定在``STDOUT``上。这个包可以包含一个镜像的所有父层，以及其所有带标签的版本，也可以限定其标签。

```bash
docker save [OPTIONS] IMAGE [IMAGE...]
```

选项：

|Name, shorthand|Default|Description|
|:--|:--|:--|
|--output , -o||Write to a file, instead of STDOUT|

```bash
docker save busybox > busybox.tar
docker save --output busybox.tar busybox
docker save -o fedora-all.tar fedora
docker save -o fedora-latest.tar fedora:latest
```

上述命令都是写入tar包，但是第一条是将标准输出流写入文件。

也可以同时打包成gz:

```bash
docker save myimage:latest | gzip > myimage_latest.tar.gz
```

或者选择标签：

```bash
docker save -o ubuntu.tar ubuntu:lucid ubuntu:saucy
```

### docker load

从一个tar（可以是被压缩的gzip，bzip2，或者xz）或者标准输入流中载入一个镜像。会复原原本的镜像和其标签。

```bash
docker load [OPTIONS]
```

|Name, shorthand|Default|Description|
|:--|:--|:--|
|--input , -i||Read from tar archive file, instead of STDIN|
|--quiet , -q||Suppress the load output|

例子：

```bash
docker load < busybox.tar.gz
docker load --input fedora.tar
```

### docker image save&load

``docker save``和``docker image save``基本一致；``docker load``和``docker image load``基本一致。

### docker export

``docker save``命令保存的是镜像，而``docker export``命令是将容器的文件系统打包成tar。但是``docker export``并不会将容器数据卷的内容输出，如果数据卷被mount在容器的一个文件夹中，也只会将本身就在文件夹中从内容输出。

```bash
docker export [OPTIONS] CONTAINER
```

选项：

|Name, shorthand|Default|Description|
|:--|:--|:-|
|--output , -o||Write to a file, instead of STDOUT|

和``docker save``类似：

```bash
docker export red_panda > latest.tar
docker export --output="latest.tar" red_panda
```

### docker import

引入tar或者其他来源的内容创建一个文件系统镜像。

```bash
docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
```

``URL``指向dockerhost上的任一带有文件系统或者单纯文件的archive(.tar, .tar.gz, .tgz, .bzip, .tar.xz, or .txz)。docker在import的时候会自动解包，如果是单个文件，一定要写其相对于host的绝对路径，如果是远端，则``URI``一定要以``http://``或者``https://``开头。

|Name, shorthand|Default|Description|
|:--|:--|:--|
--change , -c||Apply Dockerfile instruction to the created image|
--message , -m||Set commit message for imported image|
--platform||Set platform if server is multi-platform capable|

基本和[``docker commit``](#docker-commit)的命令类似。

从远端import

```bash
docker import https://example.com/exampleimage.tgz
```

import本地文件

```bash
cat exampleimage.tgz | docker import - exampleimagelocal:new
```

```bash
cat exampleimage.tgz | docker import --message "New image imported from tarball" - exampleimagelocal:new
```

```bash
docker import /path/to/exampleimage.tgz
```

```bash
docker import /path/to/exampleimage.tgz
```

import本地文件夹

```bash
sudo tar -c . | docker import - exampleimagedir
```

```bash
sudo tar -c . | docker import --change "ENV DEBUG=true" - exampleimagedir
```

### docker image import

``docker image import``与``docker import``相同。

## 总结

了解了上述这些，我们应该很容易从image构建container，也很容易将container的改变传到image，在没有涉及到dockerfile的情况下，我们已经完全可以构建并且运行docker应用了。
