---
title: ArgParser的使用
date: 2022-07-13 13:35:25
tags:
- python
- argparse
categories:
- 教程
- python使用
---
命令行运行Python文件时，可以添加相应的参数。如：

```
python add.py 1 2
```

在运行时，``from sys import argv``就可以读取命令行参数，通常是一个列表，此处的 ``argv = [add.py, 1, 2]``。对于比较复杂的程序，使用这种方式读取命令行参数显然还是比较麻烦的。于是就引入了 ``argparse``模块。具体可以参考官方文档：[Parser for command-line options, arguments and sub-commands](https://docs.python.org/3/library/argparse.html "argparse官方文档")

```python
import argparse

def parse_args():
    """
    :return:进行参数的解析
    """
    description = "you should add those parameter"                   # 步骤二
    parser = argparse.ArgumentParser(description=description)        # 这些参数都有默认值，当调用parser.print_help()或者运行程序时由于参数不正确(此时python解释器其实也是调用了pring_help()方法)时，
                                                                     # 会打印这些描述信息，一般只需要传递description参数，如上。
  
    parser.add_argument("-rp", "--replay_name", default="test.zip",type=str, help="复盘名称")
    parser.add_argument("-rv", "--redview_on", action="store_true", default=False, help="是否是红方视角")
    parser.add_argument("-s", "--scene", type=int, default=20220621, help="想定编号")
    parser.add_argument("-dp", "--data_path", type=str, default="./logs/data/", help="解析数据路径")
    parser.add_argument("-sc", "--scope_on", action="store_true", default=False, help="是否解析视野")
  
    parser.print_help()
    return parser.parse_args()

if __name__ == "__main__":
    args = parse_args()
```

先给出一个例子。具体的流程是：

+ 先实例化一个 ``ArgumentParser``可以添加描述。

  + 官方文档中还有其余的属性
    ![ArgumentParser](https://by3302files.storage.live.com/y4mJNjqUbkRtw0yibJEHVpamztvolm22ce2nFuZtxVZ1VgibrF8mFt7uTu8S6X0bUwu4MLGj3rt8bi2poj-A9BdaCKfHXgxhelLV3cagIaCpB8xdqqysD-zGisjV2LvIlWMmRYJtE9MM9EezEcqe7wOk5HP9UGGuh-GqBYm6Tv2uxBrieCXs9v3tuQEyo7aaU-J?width=817&height=524&cropmode=none "ArgumentParser属性")

+ 然后调用 ``add_argument``方法添加命令行选项
  ![add_argument](https://by3302files.storage.live.com/y4maWV3d07DQSL5IGeVCVf7D4vdZuNyY-C-vBuLRdFcLDnlqzGs-ZJOWvZn1knE5MzNXk4jqMItmQncI6fsGnNycdxUQ-oeUzaTMmCe6xgn7rorPiraIFop-iOfNICje333kKrZR4TTAAhYr6sdoHoNQs0kR6kAAb95R4RddtSjjkbY5als1fbhwmMCQFEo6VH2?width=802&height=491&cropmode=none "add_argument方法 ")

  + name or flags用于识别参数选项，其中有-或者--前缀的是可选命令，如果不加前缀则是位置参数（positional argument) 如运行需要的文件名。

    ```python-repl
    >>> parser = argparse.ArgumentParser(prog='PROG')
    >>> parser.add_argument('-f', '--foo')
    >>> parser.add_argument('bar')
    >>> parser.parse_args(['BAR'])
    Namespace(bar='BAR', foo=None)
    >>> parser.parse_args(['BAR', '--foo', 'FOO'])
    Namespace(bar='BAR', foo='FOO')
    >>> parser.parse_args(['--foo', 'FOO'])
    usage: PROG [-h] [-f FOO] bar
    PROG: error: the following arguments are required: bar
    ```
  + action是指接受参数之后要执行的动作，通常是 ``store_false``，``store_true``或者 ``store_const``，不需要接受其余参数，直接将对应的选项变量保存为特定的值。这是使用得比较多的选项。``append``是一个特殊的action，它会将参数追加到列表中。``count``是一个特殊的action，它会将参数计数，并将计数值保存到变量中。
  + 其他还有一些方法这里就不再赘述了，具体使用的时候可以参考官方文档。
+ 最后调用 ``parse_args``方法解析命令行参数，返回一个 ``Namespace``对象，其中包含了所有的参数。
