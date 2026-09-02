---
title: WSL安装及用miniconda创建python3.7环境
date: 2021-02-07 15:11:57
tags:
- WSL
- Linux
categories:
- 教程
- Linux使用
---
WSL在windows上的确相当的鸡肋，但是假若没有配置服务器，WSL配合VSCode，也可以充当生产力工具。比较大的bug是，WSL的文件与NTFS的文件系统并不同步，可能是创建了多个缓存，因此需要不断地Reload来更新。如果有服务器资源或者Linux桌面版，那么配合VSCode远程开发工具就已经很香了。以下是WSL的安装教程。

<!-- more -->

### 1. WSL准备

参考文章：[vscode wsl入门 - tnnmigga的文章 - 知乎](https://zhuanlan.zhihu.com/p/104060131)

+ 首先要在windows打开linux子系统功能
  控制面板——程序——启用或关闭windows功能——适用于linux的windows子系统
+ 然后重启电脑
+ 打开win10应用商店，搜索Ubuntu
+ 下载安装好后启动，可以看到Ubuntu bash， 几分钟后初始化完成（这里没有提示，几分钟后按Enter键）即可。输入用户名: dison （不能有大写字母，这里dison用于用户指代，可以输自己的名字）及密码。
+ `<u>`如果不喜欢用Ubuntu自带的bash的话可以使用Pycharm或者vscode的terminal，方便代码的复制粘贴。直接在terminal中输入wsl或者Ubuntu就可以打开Ubuntu子系统 `</u>`
+ 换源

  + 获取权限

    ```bash
    sudo chown -R dison /etc/apt/sources.list
    ```
  + 备份原文件

    ```bash
    sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
    ```
  + 编辑源

    ```bash
    sudo vim /etc/apt/sources.list
    ```

    打开sources.list后按i键进入insert模式，将原来的源都用#注释掉。（如果会使用vim编辑器可以使用其他的快捷键）

    复制下面的源：

    ```bash
    # 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse

    # 预发布软件源，不建议启用
    # deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse

    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
    ```

  光标移到最下方右键单击粘贴源。

  按Esc键退出编辑模式，依次输入 :wq保存更改。

  + 更新源文件

  ```bash
    sudo apt update
  ```

### 2. 安装miniconda

参考文章：[在Ubuntu上安装Miniconda](https://blog.csdn.net/weixin_30486037/article/details/97982277)

+ 下载并安装

  ```bash
  sudo apt-get install wget
  wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
  bash Miniconda3-latest-Linux-x86_64.sh
  ```

  ```bash
  # 一直按回车然后输入yes
  please answer 'yes' or 'no':
  >>> yes

  # 选择安装路径, 文件名前加点号表示隐藏文件
  Miniconda3 will now be installed into this location:
  >>> ~/.miniconda3

  # 添加配置信息到 ~/.bashrc文件
  Do you wish the installer to initialize Miniconda3 by running conda init? [yes|no]
  [no] >>> yes
  ```
+ 运行配置信息文件

  ```bash
  source ~/.bashrc
  ```
+ 测试是否安装成功

  ```bash
  conda --version
  ```
+ 如果是服务器用户，可以在Miniconda文件夹同目录下创建.condarc文件增加如下channel：

  ```json
  channels:
    - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
    - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
    - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
    - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
  ssl_verify: true

  ```

  也可以用如下命令添加源。

  ```bash
  conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
  ```

### 3. python3.7及相关包安装

+ 如果已经进入了base环境可以用如下命令退出环境

  ```bash
  conda deactivate
  ```
+ 创建python3.7环境

  ```bash
  conda create -n venv python=3.7
  ```
+ 激活虚拟环境

  ```bash
  conda activate venv
  ```
+ 安装相关包

  ```bash
  conda install pandas
  conda install pika
  conda install pymysql
  conda install xlrd=1.2.0
  conda install matplotlib
  conda install openpyxl
  conda install pytorch torchvision torchaudio cudatoolkit=10.2 -c pytorch
  ```

### 4.运行说明

+ 路径说明

  WSL能够运行Windows系统中的文件，需要在路径前加上/mnt，比如切换到D盘即为

  ```bash
  cd /mnt/d
  ```
+ 由于.so文件使用了python3.7编译，因此在运行程序前先激活python3.7环境，然后再运行相关脚本

  ```
  conda activate venv
  ```
