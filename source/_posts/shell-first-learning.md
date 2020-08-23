---
title: 初识 Shell 脚本
date: 2020-08-23 10:00:45
tags: Shell脚本
categories: Shell
---

## 概述

Shell 是一种脚本语言，那么，就必须有解释器来执行这些脚本，常见的脚本解释器有：

- **bash：**是 Linux 标准默认的 shell。bash 由 Brian Fox 和 Chet Ramey 共同完成，是 BourneAgain Shell 的缩写，内部命令一共有 40 个。

- **sh：** 由 Steve Bourne 开发，是 Bourne Shell 的缩写，sh 是 Unix 标准默认的 shell。

另外还有：ash、 csh、 ksh 等。我们可以在终端通过以下命令来查看当前系统支持哪些 shell 类型：

```
cat /etc/shells
```

比如我的系统是 Mac OS，那么结果显示如下：

```
MoosdeMacBook-Pro:Desktop moos$ cat /etc/shells
# List of acceptable shells for chpass(1).
# Ftpd will not allow users to connect who are not using
# one of these shells.

/bin/bash
/bin/csh
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
```

Shell 经过了 POSIX 的标准化，使其可以在不同的 Linux 系统上进行移植，它属于解释性语言。本文关注的是 bash，也就是 Bourne Again Shell，由于易用和免费，Bash 在日常工作中被广泛使用。同时，Bash 也是大多数Linux 系统默认的 Shell。

Shell 的运行模式大概分为两种：

- 交互式：在**终端**直接输入命令进行执行，并等待执行完毕后再去执行下一条命令。

- 非交互式：以非交互式方式执行的shell就是运行过程中不需要与用户输入输出打交道的shell。其实就是命令的集合，将命令集合直接放在一个脚本文件中，然后由 Bash Shell 读取并执行。一般情况下运维人员会把需要处理的命令逻辑写入脚本文件中，一次执行即可。也就是实现了 Linux 的自动化运维，效率也更高。

本文中讲解的 Shell 脚本都是基于非交互式，

## 第一个 Shell 脚本

按照旧例，我们先从一个 Hello world 开始：

首先创建一个 `sh` 格式的文件 `hello_world.sh ` ：

```
touch hello_world.sh
```

接着打开该文件，并加入如下code：

```shell
#!/bin/bash
# first shell 
echo "Hello, world!"
```

程序很简单，一共就三行，第二行是注释。`#!` 是一种约定的标记，表示该脚本应该通过什么类型的解释器来执行，可以看到，我们这里使用的是 **bash** 类型解释器。那么我们的第一个脚本编写好了，应该如何执行呢？其实很简单，只需要在终端执行下面命令：

```
MoosdeMacBook-Pro:Desktop moos$ chmod +x hello_world.sh 
MoosdeMacBook-Pro:Desktop moos$ ./hello_world.sh 
Hello, world!
MoosdeMacBook-Pro:Desktop moos$ 
```

`chmod +x` 命令是为了让 hello_world.sh 拥有可执行权限，接着通过 `./hello_world.sh` 就可以成功执行我们的第一个脚本啦，可以看到终端的确打印出了预期的提示。

**echo** 命令是 Shell 的一个内部指令，用于在屏幕上打印出指定的字符串。除此以外还有 `printf` 命令，可以实现更加强大的格式化输出操作，它的用法将在后面介绍。

关于注释的问题： 在 shell 中使用 `#` 进行注释，注意，sh 里面没有多行注释，只能每一行加一个 `#` 号。

-----------

和大多数语言一样，Shell 也拥有完整的语法结构，例如变量、函数、表达式、运算符、语句块等等，其中很多规则和主流语言类似，但作为脚本语言，它也拥有一些特殊的技能，我们下面一起来了解一下。

### Shell 中的变量

为了方便，我们通过下面这个例子来看一下如何定义变量：

```sh
#!/bin/bash
player_name='Moosphon'
player_country="China"
player_score=96
echo $player_name
echo ${player_country}
player_name=Jack
echo "${player_name}'s score is ${player_score}"
```

在终端执行上述脚本的结果如下所示：

```
MoosdeMacBook-Pro:Desktop moos$ ./hello_world.sh 
Moosphon
China
Jack's score is 96
```

我们在定义变量的时候不需要为其指定具体的数据类型，常见的数据类型有整型、字符串和数组。关于变量的定义，需要注意的是，**变量名和等号之间不能有空格**，这可能和你熟悉的所有编程语言都不一样。同时，变量的命名须遵循如下规则：

- 命名只能使用英文字母，数字和下划线，首个字符不能以数字开头
- 变量名中间不能有空格，可以使用下划线
- 不能使用标点符号
- 不能使用bash里的关键字（可用 help 命令查看保留关键字）

当然，除了这些基本操作以外，Shell 还支持通过 Linux 语句来给变量赋值：

```shell
for file in `ls /Users/xx/Desktop`
或
for file in $(ls /Users/xx/Desktop)
```

顾名思义，以上是遍历桌面目录下的所有文件。另外，我们可以在 shell 中定义只读变量，即有点类似于 Java 中的 `final` 类型变量，一旦赋值以后就不能被修改其值：

```shell
#!/bin/bash
myName="Jack"
readonly myName
myName="Tom"
#会报错：/bin/sh: NAME: This variable is read only.
```

#### 字符串

字符串类型应该是最常用到的数据类型了，它可以通过单引号，双引号，甚至不用引号来定义，上方的例子中已经给出示例。单引号只能原样输出字符串，比较局限，而双引号支持格式化操作，例如添加转义符。也支持占位操作，即上面例子中在字符串中穿插具体变量，虽然是可以直接通过 `$paramName` 的方式来使用，但会引起歧义，推荐使用 `${paramName}` 的方式。这里再来针对字符串几个常见的功能需求来看一下 Shell 中该如何实：

- **获取字符串长度**

  我们可以通过以下语句来获取字符串的长度：

  ```sh
  word="friend"
  echo ${#friend} 
  #输出结果为：6
  ```

- **字符串提取**

  字符串的提取也很简单：

  ```sh
  welcome_words="Hello everyone, welcome to my kingdom!"
  echo ${welcome_words:6:8}
  #输出结果为：everyone
  ```

  其中，第一个冒号后面的数字是初始位置，而第二个冒号后的数字则是要截取的长度。

- **获取子字符串位置**

  获取字符串中某个字符的位置需要借助于 `expr` 命令来求表达式的值：

  ```sh
  welcome_words="Hello everyone, welcome to my kingdom!"
  echo ${welcome_words:6:8}
  echo `expr index "$welcome_words" e`
  #输出结果为：1
  ```

#### 数组

Shell 中同样拥有数组类型，bash 支持一维数组（不支持多维数组），并且没有限定数组的大小。与其他主流语言类似，数组元素的下标由 0 开始编号，获取数组中的元素要利用下标，下标可以是整数或算术表达式，其值应大于或等于 0。

我们可以通过如下方式来定义和使用数组：

```shell
#!/bin/bash
player_array=(Jack Tom Bob Lucy)
#or
player_array[0]=Jack
player_array[1]=Tom
player_array[2]=Bob
player_array[3]=Lucy

echo "the first player is ${player_array[0]}"
#结果：the first player is Jack
echo "All players: ${player_array[@]}"
#结果：All players：Jack Tom Bob Lucy
echo "We have ${#player_array[@]} players"
#or
echo "We have ${#player_array[*]} players"
#结果：We have 4 players
```

定义数组和其他语言类似，可以通过括号声明一个元素集合，并在内部定义元素，也可以通过指定数组下标值的形式来定义数组。值得注意的是，我们可以直接通过 `array[@]` 来获取所有元素，并可以通过 `${#array[]@}` 或者 `${#array[*]}` 来获取数组大小。

#### 变量类型

运行 shell 时，会同时存在如下三种变量：

- **局部变量** 

  局部变量在脚本或命令中定义，仅在当前 shell 实例中有效，其他 shell 启动的程序不能访问局部变量。

- **环境变量** 

  所有的程序，包括 shell 启动的程序，都能访问环境变量，有些程序需要环境变量来保证其正常运行。必要的时候 shell 脚本也可以定义环境变量。

- **shell 变量** 

  shell 变量是由 shell 程序设置的特殊变量。shell 变量中有一部分是环境变量，有一部分是局部变量，这些变量保证了 shell 的正常运行

本文之前的示例中一直都在使用局部变量，环境变量部分涉及到内容还比较多，本文篇幅有限，后续会专门针对环境变量来发文一篇，详细介绍环境变量的分类以及如何查看和配置。

### Shell 的参数传递



### Shell 中的语句块



### Shell 中的常用命令



### Shell 中的函数





 