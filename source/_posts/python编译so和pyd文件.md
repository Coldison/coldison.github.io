---
title: python编译so和pyd文件
date: 2022-07-18 13:31:23
tags:
- python
- 编译
- so
- pyd
categories:
- 教程
- python使用
---
python代码即便是编译成pyc也同样可以被反编译，通常为了产品发布跟代码保护，会编译成so(linux)或者pyd(windows)。

### 配置

主要流程是，先调用Cython包将python代码编译成c代码，再将c代码编译成so文件跟pyd文件。由于使用了gcc，所以需要安装对应的build工具。在windows平台可以通过Visual Studio Installer安装对应的包。

![Visual Studio Installer 配置C环境](https://by3302files.storage.live.com/y4mjOIpaxoglVNQtOuyJU2tGo30SMw22C61fR1HgyecUZf0la_8_eL8U8OKQzzNEggffhOxJa6SBBgy1YHwVjIFxzJJW6R1_d2N31XVSWR5BQPmwT7rwHN5hEHfuJ0G2FzDMTOB64tHaNuhqKKu1P_m3qayydJIqX-5TD4Ndr2jFtIrm85rxeL1xYuxo6Rid2si?width=1278&height=686&cropmode=none)

Linux 需要自行配置下c++生成工具。

### 运行

参考git项目[PythonCompilerC](https://github.com/HamzaAissaoui/PythonCompilerC)，一大部分工作在于对目录结构的处理，而这个项目本身生成的so/pyd文件全部在对应的build_path下，原有的目录结构消失。并且，由于不同平台的命令行格式不一样，导致编译的时候出了很多bug。对此做了一些修改，具体项目参见[PythonCompileSo](https://github.com/Coldison/PythonCompile2So)。

将本项目放到要编译的项目的目录下。注意项目中的__init__.py文件需要删掉，这部分编译会出现字符错误。

Windows平台：

```cmd
  python PROJECTPATH\scripts\compile.py --project-dir PROJECTPATH --build-lib BUILDPATH
  ```

Linux平台：

```bash
  python PROJECTPATH/scripts/compile.py --project-dir PROJECTPATH --build-lib BUILDPATH
  ```
