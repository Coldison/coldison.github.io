---
title: shell编程
tags:
- Linux
- shell
categories:
- Linux使用
- shell编程
---

[菜鸟教程shell](https://www.runoob.com/linux/linux-shell.html)

虽然对shell编程了解一点，但是毕竟还是没有系统性学过，因此没法实现自己想要的功能。

现在说一下我的需求，目前我的需求应该是修改现有的文件的一个变量，虽然其他的脚本语言也可以，但是最好是shell，因为docker容器中再配环境有一些烦人，而且这样的话可迁移性就会稍弱一点。

由于追求实用而不是全面，所以很多内容没法覆盖，等用到再学。

## shell解释器与执行

现在已经有很多shell脚本解释器了，比较常用的是

+ Bourne Shell（/usr/bin/sh或/bin/sh）
+ Bourne Again Shell（/bin/bash）

参考菜鸟教程，这里也主要实用Bash。在每个脚本的首行都要注明使用的是什么解释器：

```bash
#!/bin/bash
echo "Hello World !"
```

在写完shell脚本之后，**需要对脚本添加可执行权限**。将上述脚本保存成 ``test.sh``。然后切换到对应目录，执行：

```bash
chmod +x ./test.sh  #使脚本具有执行权限
./test.sh  #执行脚本
```

记得一定是加上 ``./``表示相对路径，直接写 ``test.sh``linux系统会从环境变量中寻找命令，而通常是没有的。

其次是可以使用解释器，文件路径是解释器的参数:

```bash
/bin/sh test.sh
sh test.sh
/bin/bash test.sh
bash test.sh
```

## shell变量

### 定义

使用等号来定义变量 ``=``，需要注意的是，``=``等号左右不能够有空格，变量的命名规则：

+ 命名只能使用英文字母，数字和下划线，首个字符不能以数字开头。
+ 中间不能有空格，可以使用下划线_。
+ 不能使用标点符号。
+ 不能使用bash里的关键字（可用help命令查看保留关键字）。

也可以在语句中隐式地为变量赋值：

```bash
for file in `ls /etc`
或
for file in $(ls /etc)
```

以上语句将 /etc 下目录的文件名循环出来。

### 从输入流中读取

```bash
read aNum
```

除了定义之外，还可以使用read关键字从输入流中读取变量。

### 使用

在变量名前加上 ``$``符号可以使用变量。

```bash
your_name="qinjx"
echo $your_name
echo ${your_name}
```

花括号是变量的定界符，在空格无法定界的时候，花括号可以帮助解释器识别变量。

```bash
for skill in Ada Coffe Action Java; do
    echo "I am good at ${skill}Script"
done
```

可以给变量进行二次赋值：

```bash
your_name="tom"
echo $your_name
your_name="alibaba"
echo $your_name
```

### 只读变量

可以用 ``readonly``声明只读变量。

```bash
#!/bin/bash
myUrl="https://www.google.com"
readonly myUrl
myUrl="https://www.runoob.com"
```

### 删除变量

可以用 ``unset``命令删除变量。

```bash
#!/bin/sh

myUrl="https://www.google.com"
unset myUrl
echo $myUrl
```

删除变量之后再 ``echo``没有任何输出也不报错。

### 变量作用域

在运行shell时，主要有三种变量：

+ 局部变量 局部变量在脚本或命令中定义，仅在当前shell实例中有效，其他shell启动的程序不能访问局部变量。
+ 环境变量 所有的程序，包括shell启动的程序，都能访问环境变量，有些程序需要环境变量来保证其正常运行。必要的时候shell脚本也可以定义环境变量。
+ shell变量 shell变量是由shell程序设置的特殊变量。shell变量中有一部分是环境变量，有一部分是局部变量，这些变量保证了shell的正常运行。

### 括号

在讲变量之前，有必要先总结一下bash中各种括号的使用。

#### 小括号()

单个小括号用来创建一个子shell，例如：

```bash
$ pwd
/home/xfeng
$ (cd /tmp; pwd)
/tmp
```

子shell允许在不影响当前shell环境下执行操作。

小括号还可以替换命令，``result=$(command)``与 ``result=`command` ``

```bash
fengxi@ubuntu:~/bash$ var=$(pwd)
fengxi@ubuntu:~/bash$ echo $var
/home/fengxi/bash
```

#### \(\(\)\)的用法

两个小括号可以进行算术运算。

```bash
fengxi@ubuntu:~/bash$ var=$(20+5)
20+5: command not found
```

双小括号具有C语言类型变量增减的功能。

```bash
fengxi@ubuntu:~/bash$ var=2
fengxi@ubuntu:~/bash$ (( var++ ))
fengxi@ubuntu:~/bash$ echo $var
3
fengxi@ubuntu:~/bash$ (( var-- ))
fengxi@ubuntu:~/bash$ echo $var
2
```

#### []的用法

单个中括号[]通常调用一个名为[的程序，你可以用man查询一下[，它的返回结果和man test的结果是一样的。[和test都是bash的builtin函数。

```bash
fengxi@ubuntu:~/bash$ var=123
fengxi@ubuntu:~/bash$ if [ $var == 123 ]; then echo yes; else echo no; fi
yes
```

在[]中，==和=效果一致。

```bash
fengxi@ubuntu:~/bash$ if [ $var = 123 ]; then echo yes; else echo no; fi
yes
```

**我自己的实验，中括号也可以进行直接进行算术运算，下面我们会详述。**

#### 双中括号的用法

双中括号不像[]是一个程序而是一个关键字。可以在其中使用逻辑运算符 ``&&``或者 ``||``，还有正则表达式匹配操作符 ``=~``。这些都不能在[]中使用。

#### {}的用法

{}主要应用在参数展开上。

+ ``${varname:-word}`` 如果varname存在且非null，则返回其值；否则，返回word
+ ``${varname:=word}`` 如果varname存在且不是null，则返回它的值；否则，设置它为word，并返回其值
+ ``${varname:?message}`` 如果varname存在且非null，则返回它的值；否则，显示varname:message，并退回当前的命令或脚本。省略message会出现默认信息parameter null or not set。用途：主要是为了捕捉由于变量未定义所导致的错误。
+ ``${varname:+word}`` 如果varname存在且非null，则返回word，否则，返回null。

此外，{}也应用在模式匹配运算符上，假设变量path的值为··/home/fengxi/long.file.name如下所示：

+ ``${variable#pattern}``如果模式匹配与变量值的开头处，则删除匹配的最短部分，并返回剩下的部分。例：\${path\#/*/} 结果为：fengxi/long.file.name
+ ``${variable##pattern}`` 如果模式匹配与变量值的开头处，则删除匹配的最长部分，并返回剩下部分。例：\${path\#\#/*/} 结果为：long.file.name
+ ``${variable%pattern}`` 如果匹配模式匹配与变量的结尾处，则删除匹配的最短部分，并返回剩下部分。例：\${path\%.*} 结果为：/home/fengxi/long.file
+ ``${variable%%pattern}`` 如果匹配模式匹配与变量的结尾处，则删除匹配的最长部分，并返回剩下的部分。例：\${path\%\%.*} 结果为：/home/fengxi/long

巧记方法：因为#在键盘上$的左侧，也就是前面，\%在键盘上\$的右侧，也就是后面。所以#匹配的头部，%匹配的是尾部。

### 变量类型

#### 字符串

除了数字类型之外，就是字符串类型，字符串是shell中最常用最有用的数据类型。字符串可以用单引号，可以用双引号，也可以不用引号。

1. 单引号

   ```bash
   str='this is a string'
   ```

   + 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的。
   + 单引号字串中不能出现单独一个的单引号（对单引号使用转义符后也不行），但可成对出现，作为字符串拼接使用。
2. 双引号

   ```bash
   your_name="runoob"
   str="Hello, I know you are \"$your_name\"! \n"
   echo -e $str
   ```

   + 双引号里可以有变量。
   + 双引号里可以出现转义字符。
3. 字符串可以拼接：

   ```bash
   your_name="runoob"
   # 使用双引号拼接
   greeting="hello, "$your_name" !"
   greeting_1="hello, ${your_name} !"
   echo $greeting  $greeting_1

   # 使用单引号拼接
   greeting_2='hello, '$your_name' !'
   greeting_3='hello, ${your_name} !'
   echo $greeting_2  $greeting_3
   ```

4. 获取字符串长度

   ```bash
   string="abcd"
   echo ${#string}   # 输出 4
   ```

   当变量为数组时， ``${#string}``等价于${#string[0]}$。

   ```bash
   string="abcd"
   echo ${#string[0]}   # 输出 4
   ```

5. 提取子字符串

   ```bash
   string="runoob is a great site"
   echo ${string:1:4} # 输出 unoo
   ```

6. 查找子字符串

   ```bash
   string="runoob is a great site"
   echo `expr index "$string" io`  # 输出 4
   ```

   注意此处是反引号 `` ` ``。

#### shell 数组

bash支持一维数组，并且没有大小限制。

1. 定义数组

   在 Shell 中，用括号来表示数组，数组元素用"空格"符号分割开。定义数组的一般形式为：

   ```bash
   array_name=(value0 value1 value2 value3)
   ```

   ```bash
   array_name=(
   value0
   value1
   value2
   value3
   )
   ```

   单独定义数组的各个分量：

   ```bash
   array_name[0]=value0
   array_name[1]=value1
   array_name[n]=valuen
   ```

   可以不使用连续的下标，而且下标的范围没有限制。
2. 读取数组

   读取数组元素值如上所述：

   ```bash
   valuen=${array_name[n]}
   ```

   使用 ``@``符号可以获取数组中的所有元素，例如：

   ```bash
   echo ${array_name[@]}
   ```

3. 获取数组的长度

   与获取字符串方法类似。

   ```bash
   # 取得数组元素的个数
   length=${#array_name[@]}
   # 或者
   length=${#array_name[*]}
   # 取得数组单个元素的长度
   lengthn=${#array_name[n]}
   ```

4. 关联数组
   bash支持关联数组，类似python的字典，可以使用任意的字符串或者整数作为下标来访问数组元素。
   需要使用 ``declare``关键字进行声明：``declare -A array_name``。
   ``-A``用于声明一个关联数组，关联数组的键是唯一的：

   ```bash
   declare -A site=(["google"]="www.google.com" ["runoob"]="www.runoob.com" ["taobao"]="www.taobao.com")
   ```

   也可以分别声明键值：

   ```bash
   declare -A site
   site["google"]="www.google.com"
   site["runoob"]="www.runoob.com"
   site["taobao"]="www.taobao.com"
   ```

   可以使用键值访问：

   ```bash
   echo ${site["google"]}
   ```

   关联数组也适用于特殊符号：

   ```bash
   declare -A site
   site["google"]="www.google.com"
   site["runoob"]="www.runoob.com"
   site["taobao"]="www.taobao.com"

   echo "数组的元素为: ${site[*]}"
   echo "数组的元素为: ${site[@]}"
   ```

   前面加上 ``!``可以访问键值：

   ```bash
   declare -A site
   site["google"]="www.google.com"
   site["runoob"]="www.runoob.com"
   site["taobao"]="www.taobao.com"

   echo "数组的键为: ${!site[*]}"
   echo "数组的键为: ${!site[@]}"
   ```

## shell注释

### 单行注释

``#``号可以完成单行注释。

### 多行注释

除了多行 ``#``号之外还可以使用：

```bash
:<<EOF
注释内容...
注释内容...
注释内容...
EOF
```

还可以:

```bash
:<<'
注释内容...
注释内容...
注释内容...
'

:<<!
注释内容...
注释内容...
注释内容...
!
```

``'``和 ``!``相当于定界符。

## shell传递参数

与python类似，shell脚本可以接受参数，其中脚本文件名本身是 ``$0``，接下来第n个参数就可以用 ``$n``指代。

```bash
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

echo "Shell 传递参数实例！";
echo "执行的文件名：$0";
echo "第一个参数为：$1";
echo "第二个参数为：$2";
echo "第三个参数为：$3";
```

还有一些跟参数相关的特殊字符：

|参数说明|说明|
|:--|:--|
|\$#| 传递到脚本的参数个数|
| \$\*| 以一个单字符串显示所有向脚本传递的参数。如"\$\*"用"括起来的情况、以"\$1 \$2 … \$n"的形式输出所有参数。|
| \$\$|脚本运行的当前进程ID号|
| \$!| 后台运行的最后一个进程的ID号|
|\$@ | 与\$\*相同，但是使用时加引号，并在引号中返回每个参数。如"\$@"用"括起来的情况、以"\$1" "\$2" … "\$n"的形式输出所有参数。|
| \$\-| 显示Shell使用的当前选项，与set命令功能相同。|
| \$?| 显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。 |

```bash
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

echo "Shell 传递参数实例！";
echo "第一个参数为：$1";

echo "参数个数为：$#";
echo "传递的参数作为一个字符串显示：$*";
```

``$*``与 ``$@``区别：

+ 相同点：都是引用所有参数。
+ 不同点：只有在双引号中体现出来。假设在脚本运行时写了三个参数 1、2、3，，则 " * " 等价于 "1 2 3"（传递了一个参数），而 "@" 等价于 "1" "2" "3"（传递了三个参数）。

## shell基本运算符

原生bash不支持简单的数学运算，主要借助于其他命令实现。例如 ``awk``和 ``expr``，``expr``最常用，``expr``是一款表达式计算工具，使用它能完成表达式的求值操作。注意两点：

+ 表达式和运算符之间要有空格，例如 ``2+2`` 是不对的，必须写成 ``2 + 2``，这与我们熟悉的大多数编程语言不一样。
+ 完整的表达式要被 `` ` ` `` 包含，注意这个字符不是常用的单引号，在 ``Esc``键下边。

```bash
#!/bin/bash

val=`expr 2 + 2`
echo "两数之和为 : $val"
```

代码中的 ``[]``执行基本的

### 算术运算符

假定 ``a=10``，``b=20``。

| 运算符 | 说明   | 举例                                                            |
| :----- | :----- | :-------------------------------------------------------------- |
| +      | 加法   | `` `expr $a + $b` ``结果为30。                                  |
| -      | 减法   | `` `expr $a - $b` ``结果为-10。                                 |
| *      | 乘法   | `` `expr $a \* $b` ``结果为200。注意这里的乘法要使用转义字符。  |
| /      | 除法   | `` `expr $b / $a` ``结果为2。                                   |
| %      | 取余   | `` `expr $b % $a` ``结果为0。                                   |
| =      | 赋值   | ``a=$b``把变量b的值赋给a。                                      |
| ==     | 相等   | 用于比较两个数字，相同则返回true。``[ $a == $b ]``返回 false。  |
| !=     | 不相等 | 用于比较两个数字，不相同则返回true。``[ $a != $b ]``返回 true。 |

```bash
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

a=10
b=20

val=`expr $a + $b`
echo "a + b : $val"

val=`expr $a - $b`
echo "a - b : $val"

val=`expr $a \* $b`
echo "a * b : $val"

val=`expr $b / $a`
echo "b / a : $val"

val=`expr $b % $a`
echo "b % a : $val"

if [ $a == $b ]
then
   echo "a 等于 b"
fi
if [ $a != $b ]
then
   echo "a 不等于 b"
fi
```

在MAC的shell中，``expr``的语法是 ``$((表达式))``，``*``也不需要转义字符。

### 关系运算符

同样假定 ``a=10``，``b=20``。

| 运算符 | 说明                                                  | 举例                        |
| :----- | :---------------------------------------------------- | :-------------------------- |
| -eq    | 检测两个数是否相等，相等返回 true。                   | [\$a -eq \$b ] 返回 false。 |
| -ne    | 检测两个数是否不相等，不相等返回 true。               | [\$a -ne \$b ] 返回 true。  |
| -gt    | 检测左边的数是否大于右边的，如果是，则返回 true。     | [\$a -gt \$b ] 返回 false。 |
| -lt    | 检测左边的数是否小于右边的，如果是，则返回 true。     | [\$a -lt \$b ] 返回 true。  |
| -ge    | 检测左边的数是否大于等于右边的，如果是，则返回 true。 | [\$a -ge \$b ] 返回 false。 |
| -le    | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | [\$a -le \$b ] 返回 true。  |

```bash
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

a=10
b=20

if [ $a -eq $b ]
then
   echo "$a -eq $b : a 等于 b"
else
   echo "$a -eq $b: a 不等于 b"
fi
if [ $a -ne $b ]
then
   echo "$a -ne $b: a 不等于 b"
else
   echo "$a -ne $b : a 等于 b"
fi
if [ $a -gt $b ]
then
   echo "$a -gt $b: a 大于 b"
else
   echo "$a -gt $b: a 不大于 b"
fi
if [ $a -lt $b ]
then
   echo "$a -lt $b: a 小于 b"
else
   echo "$a -lt $b: a 不小于 b"
fi
if [ $a -ge $b ]
then
   echo "$a -ge $b: a 大于或等于 b"
else
   echo "$a -ge $b: a 小于 b"
fi
if [ $a -le $b ]
then
   echo "$a -le $b: a 小于或等于 b"
else
   echo "$a -le $b: a 大于 b"
fi
```

### 布尔运算符

``a=10``，``b=20``。

| 运算符 | 说明                                                | 举例                                      |
| :----- | :-------------------------------------------------- | :---------------------------------------- |
| !      | 非运算，表达式为 true 则返回 false，否则返回 true。 | [ ! false ] 返回 true。                   |
| -o     | 或运算，有一个表达式为 true 则返回 true。           | [\$a -lt 20 -o \$b -gt 100 ] 返回 true。  |
| -a     | 与运算，两个表达式都为 true 才返回 true。           | [\$a -lt 20 -a \$b -gt 100 ] 返回 false。 |

```bash
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

a=10
b=20

if [ $a != $b ]
then
   echo "$a != $b : a 不等于 b"
else
   echo "$a == $b: a 等于 b"
fi
if [ $a -lt 100 -a $b -gt 15 ]
then
   echo "$a 小于 100 且 $b 大于 15 : 返回 true"
else
   echo "$a 小于 100 且 $b 大于 15 : 返回 false"
fi
if [ $a -lt 100 -o $b -gt 100 ]
then
   echo "$a 小于 100 或 $b 大于 100 : 返回 true"
else
   echo "$a 小于 100 或 $b 大于 100 : 返回 false"
fi
if [ $a -lt 5 -o $b -gt 100 ]
then
   echo "$a 小于 5 或 $b 大于 100 : 返回 true"
else
   echo "$a 小于 5 或 $b 大于 100 : 返回 false"
fi
```

### 逻辑运算符

``a=10``，``b=20``。

| 运算符 | 说明       | 举例                                        |
| :----- | :--------- | :------------------------------------------ |
| ``&&`` | 逻辑的 AND | \[\[ \$a -lt 100 \&\& \$b -gt 100 \]\] 返回 false  |
| ``\|\|`` | 逻辑的 OR  | \[ \[\$a -lt 100 \|\| \$b -gt 100 \]\] 返回 true |

```bash
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

a=10
b=20

if [[ $a -lt 100 && $b -gt 100 ]]
then
   echo "返回 true"
else
   echo "返回 false"
fi

if [[ $a -lt 100 || $b -gt 100 ]]
then
   echo "返回 true"
else
   echo "返回 false"
fi
```

### 字符串运算符

``a=abc``，``b=efg``。

| 运算符                                                         | 说明                                         | 举例                      |
| :------------------------------------------------------------- | :------------------------------------------- | :------------------------ |
| =                                                              | 检测两个字符串是否相等，相等返回 true。      | [\$a = \$b ] 返回 false。 |
| !=                                                             | 检测两个字符串是否不相等，不相等返回 true。  | [\$a != \$b ] 返回 true。 |
| -z                                                             | 检测字符串长度是否为0，为0返回 true。        | [ -z \$a ] 返回 false。    |
| -n                                                             | 检测字符串长度是否不为 0，不为 0 返回 true。 | [ -n "$a" ] 返回 true。   |
| \$|检测字符串是否不为空，不为空返回 true。|[ \$a ] 返回 true。 |                                              |                           |

```bash
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

a="abc"
b="efg"

if [ $a = $b ]
then
   echo "$a = $b : a 等于 b"
else
   echo "$a = $b: a 不等于 b"
fi
if [ $a != $b ]
then
   echo "$a != $b : a 不等于 b"
else
   echo "$a != $b: a 等于 b"
fi
if [ -z $a ]
then
   echo "-z $a : 字符串长度为 0"
else
   echo "-z $a : 字符串长度不为 0"
fi
if [ -n "$a" ]
then
   echo "-n $a : 字符串长度不为 0"
else
   echo "-n $a : 字符串长度为 0"
fi
if [ $a ]
then
   echo "$a : 字符串不为空"
else
   echo "$a : 字符串为空"
fi
```

### 文件测试运算符

文件测试运算符用于检测Unix文件的各种属性。

| 操作符  | 说明                                                                        | 举例                      |
| :------ | :-------------------------------------------------------------------------- | :------------------------ |
| -b file | 检测文件是否是块设备文件，如果是，则返回 true。                             | [ -b $file ] 返回 false。 |
| -c file | 检测文件是否是字符设备文件，如果是，则返回 true。                           | [ -c $file ] 返回 false。 |
| -d file | 检测文件是否是目录，如果是，则返回 true。                                   | [ -d $file ] 返回 false。 |
| -f file | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 | [ -f $file ] 返回 true。  |
| -g file | 检测文件是否设置了 SGID 位，如果是，则返回 true。                           | [ -g $file ] 返回 false。 |
| -k file | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。                 | [ -k $file ] 返回 false。 |
| -p file | 检测文件是否是有名管道，如果是，则返回 true。                               | [ -p $file ] 返回 false。 |
| -u file | 检测文件是否设置了 SUID 位，如果是，则返回 true。                           | [ -u $file ] 返回 false。 |
| -r file | 检测文件是否可读，如果是，则返回 true。                                     | [ -r $file ] 返回 true。  |
| -w file | 检测文件是否可写，如果是，则返回 true。                                     | [ -w $file ] 返回 true。  |
| -x file | 检测文件是否可执行，如果是，则返回 true。                                   | [ -x $file ] 返回 true。  |
| -s file | 检测文件是否为空（文件大小是否大于0），不为空返回 true。                    | [ -s $file ] 返回 true。  |
| -e file | 检测文件（包括目录）是否存在，如果是，则返回 true。                         | [ -e $file ] 返回 true。  |
| -S file | 判断文件是否socket                                                          |                           |
| -L file | 判断文件是否存在并且是一个符号链接                                          |                           |

```bash
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

file="/var/www/runoob/test.sh"
if [ -r $file ]
then
   echo "文件可读"
else
   echo "文件不可读"
fi
if [ -w $file ]
then
   echo "文件可写"
else
   echo "文件不可写"
fi
if [ -x $file ]
then
   echo "文件可执行"
else
   echo "文件不可执行"
fi
if [ -f $file ]
then
   echo "文件为普通文件"
else
   echo "文件为特殊文件"
fi
if [ -d $file ]
then
   echo "文件是个目录"
else
   echo "文件不是个目录"
fi
if [ -s $file ]
then
   echo "文件不为空"
else
   echo "文件为空"
fi
if [ -e $file ]
then
   echo "文件存在"
else
   echo "文件不存在"
fi
```

## shell echo命令

``echo``命令用于输出字符串。

### 显示普通字符串

```bash
echo "it is a test"
```

如前所属，不加双引号也可。

```bash
echo it is a test
```

### 显示转义字符

双引号可以显示转义字符。

```bash
echo "\"it is a test\""
>> "it is a test"
```

### 显示变量

``read``命令从标准输入中读取一行,并把输入行的每个字段的值指定给 ``shell``变量。

```bash
#!/bin/sh
read name 
echo "$name It is a test"
```

### 显示换行

```bash
echo -e "OK! \n" # -e 开启转义
echo "It is a test"
```

### 显示不换行

```bash
#!/bin/sh
echo -e "OK! \c" # -e 开启转义 \c 不换行
echo "It is a test"
```

### 显示结果定向至文件

```bash
echo "It is a test" > myfile
```

### 显示命令执行结果

```bash
echo '$name\"'
```

## shell printf命令

``printf``也是一个输出命令，模仿C Library的 ``printf``，由POSIX标准定义，所以使用 ``printf``可移植性比较好。

``printf``使用引用文本或空格分隔的参数，外面可以在 ``printf``中使用格式化字符串，还可以指定字符串的宽度，左右对齐方式等。默认的 ``printf``不会像 ``echo``自行添加换行符，我们可以手动添加 ``\n``。

语法：

```bash
printf  format-string  [arguments...]
```

+ format-string为格式控制字符串
+ arguments为参数列表

```bash
$ echo "Hello, Shell"
> Hello, Shell
$ printf "Hello, Shell\n"
> Hello, Shell
```

python也有格式化字符串，可以很好地进行格式化输出：

```bash
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com
 
printf "%-10s %-8s %-4s\n" 姓名 性别 体重kg  
printf "%-10s %-8s %-4.2f\n" 郭靖 男 66.1234
printf "%-10s %-8s %-4.2f\n" 杨过 男 48.6543
printf "%-10s %-8s %-4.2f\n" 郭芙 女 47.9876
```

运行结果：

```bash
姓名     性别   体重kg
郭靖     男      66.12
杨过     男      48.65
郭芙     女      47.99
```

``%s %c %d %f``是格式替代符，``%s``输出字符串，``%d``整型输出，``%c``输出单个字符，``%f``输出浮点数。

``%-10s``，``-``左对齐，没有则是右对齐，表示输出一个宽度为10的字符，如果内容不足则以空格填充，超过也会全部显示出来。

``%-4.2f``中的 ``.2``指保留2位小数 。

```bash
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com
 
# format-string为双引号
printf "%d %s\n" 1 "abc"

# 单引号与双引号效果一样
printf '%d %s\n' 1 "abc"

# 没有引号也可以输出
printf %s abcdef

# 格式只指定了一个参数，但多出的参数仍然会按照该格式输出，format-string 被重用
printf %s abc def

printf "%s\n" abc def

printf "%s %s %s\n" a b c d e f g h i j

# 如果没有 arguments，那么 %s 用NULL代替，%d 用 0 代替
printf "%s and %d \n"
```

输出为

```bash
1 abc
1 abc
abcdefabcdefabc
def
a b c
d e f
g h i
j
 and 0
```

### printf的转义序列

| 序列  | 说明                                                                                                                                                                         |
| :---- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| \a    | 警告字符，通常为ASCII的BEL字符                                                                                                                                               |
| \b    | 后退                                                                                                                                                                         |
| \c    | 抑制（不显示）输出结果中任何结尾的换行字符（只在%b格式指示符控制下的参数字符串中有效），而且，任何留在参数里的字符、任何接下来的参数以及任何留在格式字符串中的字符，都被忽略 |
| \f    | 换页（formfeed）                                                                                                                                                             |
| \n    | 换行                                                                                                                                                                         |
| \r    | 回车（Carriage return）                                                                                                                                                      |
| \t    | 水平制表符                                                                                                                                                                   |
| \v    | 垂直制表符                                                                                                                                                                   |
| \\\\  | 一个字面上的反斜杠字符                                                                                                                                                       |
| \ddd  | 表示1到3位数八进制值的字符。仅在格式字符串中有效                                                                                                                             |
| \0ddd | 表示1到3位的八进制值字符                                                                                                                                                     |

实例：

```bash
$ printf "a string, no processing:<%s>\n" "A\nB"
a string, no processing:<A\nB>

$ printf "a string, no processing:<%b>\n" "A\nB"
a string, no processing:<A
B>

$ printf "www.runoob.com \a"
www.runoob.com $                  #不换行
```

## shell test命令

Shell中的 test 命令用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试。基本上跟我们前述的命令一致。

### 数值测试

| 参数 | 说明           |
| :--- | :------------- |
| -eq  | 等于则为真     |
| -ne  | 不等于则为真   |
| -gt  | 大于则为真     |
| -ge  | 大于等于则为真 |
| -lt  | 小于则为真     |
| -le  | 小于等于则为真 |

例子：

```bash
num1=100
num2=100
if test $[num1] -eq $[num2]
then
    echo '两个数相等！'
else
    echo '两个数不相等！'
fi
```

### 字符串测试

| 参数      | 说明                     |
| :-------- | :----------------------- |
| =         | 等于则为真               |
| !=        | 不相等则为真             |
| -z 字符串 | 字符串的长度为零则为真   |
| -n 字符串 | 字符串的长度不为零则为真 |

```bash
num1="ru1noob"
num2="runoob"
if test $num1 = $num2
then
    echo '两个字符串相等!'
else
    echo '两个字符串不相等!'
fi
```

### 文件测试

| 参数      | 说明                                 |
| :-------- | :----------------------------------- |
| -e 文件名 | 如果文件存在则为真                   |
| -r 文件名 | 如果文件存在且可读则为真             |
| -w 文件名 | 如果文件存在且可写则为真             |
| -x 文件名 | 如果文件存在且可执行则为真           |
| -s 文件名 | 如果文件存在且至少有一个字符则为真   |
| -d 文件名 | 如果文件存在且为目录则为真           |
| -f 文件名 | 如果文件存在且为普通文件则为真       |
| -c 文件名 | 如果文件存在且为字符型特殊文件则为真 |
| -b 文件名 | 如果文件存在且为块特殊文件则为真     |

```bash
cd /bin
if test -e ./bash
then
    echo '文件已存在!'
else
    echo '文件不存在!'
fi
```

### 测试条件连接

测试条件也可以使用 ``-a``，``-o``，``!``连接。优先级顺序是 ``!``，``-a``，``-o``，和通常的一致。

```bash
cd /bin
if test -e ./notFile -o -e ./bash
then
    echo '至少有一个文件存在!'
else
    echo '两个文件都不存在'
fi
```

## shell流程控制

### if else

```bash
if condition
then
    command1 
    command2
    ...
    commandN 
fi
```

也可以写成一行，用分号分隔，通常用于终端输入：

```bash
if [ $(ps -ef | grep -c "ssh") -gt 1 ]; then echo "true"; fi
```

``fi``关键字是对 ``if``的结束。

加上 ``else``关键字：

```bash
if condition
then
    command1 
    command2
    ...
    commandN
else
    command
fi
```

加上 ``elif``关键字：

```bash
if condition1
then
    command1
elif condition2 
then 
    command2
else
    commandN
fi
```

条件语句除了中括号 ``[]``执行之外，还可以使用 ``test``语句。

### for循环

``for``循环的一般形式：

```bash
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done
```

也可以写成一行：

```bash
for var in item1 item2 ... itemN; do command1; command2… done;
```

``in``列表包含替换、字符串和文件名。

### while 语句

``while``语句的语法格式是：

```bash
while condition
do 
   command
done
```

例子：

```bash
#!/bin/bash
int=1
while(( $int<=5 ))
do
    echo $int
    let "int++"
done
```

``let``命令是BASH中用于计算的工具，用于执行一个或者多个表达式，变量计算中不需要加上 $ 来表示变量。如果表达式中包含了空格或其他特殊字符，则必须引起来。

``while``循环可以接收键盘输入，在下面的例子中，输入信息被设置为变量 ``FILM``，按 ``Ctrl+D``结束循环。

```bash
echo '按下 <CTRL-D> 退出'
echo -n '输入你最喜欢的网站名: '
while read FILM
do
    echo "是的！$FILM 是一个好网站"
done
```

如果不写条件或者条件为真，则是无限循环。

```bash
while :
do
    command
done
```

```bash
while true
do
    command
done
```

### until循环

一般而言，while循环比较好用，但是有时候也可以使用until循环。until循环的逻辑与while刚好相反，为执行一系列命令直到条件为真为止。

```bash
until condition
do
    command
done
```

实例：

```bash
until [ ! $a -lt 10 ]
do
   echo $a
   a=`expr $a + 1`
done
```

### case ... easc

``case ... easc``为多选择语句，与其他语言的 ``switch ... case``类似，是一种多分枝选择结构，每个case分支用右圆括号开始，用两个分号 ``;;``表示break，也就是执行结束，跳出整个 ``case ... easc``语句。语法格式如下：

```bash
case 值 in
模式1)
    command1
    command2
    ...
    commandN
    ;;
模式2)
    command1
    command2
    ...
    commandN
    ;;
esac
```

例子：

```bash
echo '输入 1 到 4 之间的数字:'
echo '你输入的数字为:'
read aNum
case $aNum in
    1)  echo '你选择了 1'
    ;;
    2)  echo '你选择了 2'
    ;;
    3)  echo '你选择了 3'
    ;;
    4)  echo '你选择了 4'
    ;;
    *)  echo '你没有输入 1 到 4 之间的数字'
    ;;
esac
```

注意 ``*``号是通配符，只要是字符串就可以。

```bash
#!/bin/sh

site="runoob"

case "$site" in
   "runoob") echo "菜鸟教程"
   ;;
   "google") echo "Google 搜索"
   ;;
   "taobao") echo "淘宝网"
   ;;
esac
```

### 跳出循环

``break``命令跳出所有循环，而 ``continue``关键字跳出当前循环。

```bash
#!/bin/bash
while :
do
    echo -n "输入 1 到 5 之间的数字:"
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的! 游戏结束"
            break
        ;;
    esac
done
```

其中 ``read``关键字表示输入一个变量。当输入不在其中时，程序结束。

```bash
#!/bin/bash
while :
do
    echo -n "输入 1 到 5 之间的数字: "
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的!"
            continue
            echo "游戏结束"
        ;;
    esac
done
```

程序永远不会结束。

## shell函数

### shell函数格式

shell当然也可以运行函数，定义格式如下：

```bash
[ function ] funname [()]

{

    action;

    [return int;]

}
```

+ 可以带上 ``[function]``，也可以不加。
+ ``[return ;]``如果不加，则以最后一条命令结果作为返回值。

```bash
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

demoFun(){
    echo "这是我的第一个 shell 函数!"
}
echo "-----函数开始执行-----"
demoFun
echo "-----函数执行完毕-----"
```

带有return语句的函数：

```bash
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

funWithReturn(){
    echo "这个函数会对输入的两个数字进行相加运算..."
    echo "输入第一个数字: "
    read aNum
    echo "输入第二个数字: "
    read anotherNum
    echo "两个数字分别为 $aNum 和 $anotherNum !"
    return $(($aNum+$anotherNum))
}
funWithReturn
echo "输入的两个数字之和为 $? !"
```

函数返回值在调用该函数后通过 ``$?``来获得，``$?``只能获得上一条命令的输出，所以必须马上调用。

**注意：所有函数在使用前必须定义。这意味着必须将函数放在脚本开始部分，直至shell解释器首次发现它时，才可以使用。调用函数仅使用其函数名即可。**

### 函数参数

在调用函数时，可以向其中传入一个参数列表。在函数体内部，使用 ``$1``来表示第一个参数，注意10及以上要使用花括号访问，应该是灭有办法识别时字符串还是一个数字，所以需要使用花括号定界。示例：

```bash
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

funWithParam(){
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    echo "第十个参数为 $10 !"
    echo "第十个参数为 ${10} !"
    echo "第十一个参数为 ${11} !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"
}
funWithParam 1 2 3 4 5 6 7 8 9 34 73
```

``$``特殊命令列表参见shell参数。

## shell输入输出重定向

大多数 UNIX 系统命令从你的终端接受输入并将所产生的输出发送回​​到您的终端。一个命令通常从一个叫标准输入的地方读取输入，默认情况下，这恰好是你的终端。同样，一个命令通常将其输出写入到标准输出，默认情况下，这也是你的终端。

命令|说明
:--|:--
command > file|将输出重定向到 file。
command < file|将输入重定向到 file。
command >> file|将输出以追加的方式重定向到 file。
n > file|将文件描述符为 n 的文件重定向到 file。
n >> file|将文件描述符为 n 的文件以追加的方式重定向到 file。
n >& m|将输出文件 m 和 n 合并。
n <& m|将输入文件 m 和 n 合并。
<< tag|将开始标记 tag 和结束标记 tag 之间的内容作为输入。

这里的文件描述符：**文件描述符 0 通常是标准输入（STDIN），1 是标准输出（STDOUT），2 是标准错误输出（STDERR）。**

### 输入重定向

```bash
command > file
```

将命令的输出重定向到文件中。

```bash
who > users
```

重定向输出会覆盖文件。如果不希望内容被覆盖，可以使用
``>>``追加到文件末尾。

### 输出重定向

```bash
command < file
```

将本来从键盘得到的输入，重定向到从文件中获取。

### 带上文件描述符

前述在unix系统中，0是STDIN，1是STDOUT，2时STDERR。

#### 重定向错误描述符

```bash
command 2>file
```

追加到文件

```bash
command 2>> file
```

#### stdout和stderr合并输出

```bash
command > file 2>&1
command >> file 2>&1
```

#### 重定向输入输出

```bash
command < file1 >file2
```

### Here Document

Here Document 是 Shell 中的一种特殊的重定向方式，用来将输入重定向到一个交互式 Shell 脚本或程序。

它的基本的形式如下：

```bash
command << delimiter
    document
delimiter
```

它的作用是将两个 delimiter 之间的内容(document) 作为输入传递给 command。

+ 结尾的delimiter 一定要顶格写，前面不能有任何字符，后面也不能有任何字符，包括空格和 tab 缩进。
+ 开始的delimiter前后的空格会被忽略掉。

```bash
$ wc -l << EOF
    欢迎来到
    菜鸟教程
    www.runoob.com
EOF
3          # 输出结果为 3 行
$
```

```bash
cat << EOF
欢迎来到
菜鸟教程
www.runoob.com
EOF
```

前述的注释也是用同样的内联文档的形式完成的。

### ``/dev/null``文件

``/dev/null``是一个特殊的文件，写入到它的内容都会被丢弃；如果尝试从该文件读取内容，那么什么也读不到。但是``/dev/null``文件非常有用，将命令的输出重定向到它，会起到"禁止输出"的效果。

```bash
command > /dev/null 2>&1
```

禁止STDOUT和STDERR。

+ 这里的``2``和``>``之间不可以有空格，``2>``是一体的时候才表示错误输出。

## shell文件包含

Shell 也可以包含外部脚本。这样可以很方便的封装一些公用的代码作为一个独立的文件。

Shell 文件包含的语法格式如下：

```bash
. filename   # 注意点号(.)和文件名中间有一空格

或

source filename
```

创建第一个文件``test1.sh``：

```bash
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

url="http://www.runoob.com"
```

在第二个文件中包含：

```bash
#!/bin/bash
# author:菜鸟教程
# url:www.runoob.com

#使用 . 号来引用test1.sh 文件
. ./test1.sh

# 或者使用以下包含文件代码
# source ./test1.sh

echo "菜鸟教程官网地址：$url"
```

执行后即可输出url。
