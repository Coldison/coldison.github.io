---
title: Hexo的安装及使用
date: 2021-02-07 11:45:32
tags: 
- Hexo
- fluid
categories: 
- 教程
- 博客设置

---

本文主要介绍一下Hexo的安装与使用流程，以免自己忘记。

<!-- more -->

### 一、Hexo的安装

#### 1. Hexo的安装

需要先安装nodejs以及git。

```bash
npm install -g hexo-cli
```

#### 2. Hexo的初始化

Hexo安装完成后，在指定文件夹下初始化。

```bash
hexo init <folder>
cd <folder>
npm install
```

_config.yml下有网站的配置信息，可以在此配置大部分参数。具体配置及相关文档参见：[Hexo文档](https://hexo.io/zh-cn/docs/)

在github上创建名为```username.github.io```的仓库，在_config.yml文件的deploy部分配置自己的git仓库地址，将分支改为master。

```yaml
deploy:
  type: git
  repo: https://github.com/Coldison/coldison.github.io.git
  branch: master
```

### 二、Fluid主题配置

我选择的是fluid主题，参见[Fluid 主题文档](https://hexo.fluid-dev.com/docs/start/#%E4%B8%BB%E9%A2%98%E7%AE%80%E4%BB%8B)。

将Fluid主题文件解压，放到themes目录下并且改名fluid。在```_config.yml```中将theme改为fluid。然后将fluid文件夹下的 ```_config.yml```改名为```_config.fluid.yml```，复制到```_config.yml```同级文件夹。然后就可以在```_config.fluid.yml```修改相应的style了。

主要是修改配色，链接以及文本。我选择了偏灰的颜色，超链接是紫色系，具体可以自己修改。

### 三、Hexo命令

写作：

```bash
hexo new [layout] <title>
```

主要有post，page，draft三种，路径都是source。

也可以自己创建模板。

常用命令：

```bash
hexo g #完整命令为hexo generate,用于生成静态文件
hexo s #完整命令为hexo server,用于启动服务器，主要用来本地预览
hexo d #完整命令为hexo deploy,用于将本地文件发布到github上
hexo n #完整命令为hexo new,用于新建一篇文章
hexo g -d #两个命令的合成，一般在修改或者添加博文后直接使用这个命令
```

