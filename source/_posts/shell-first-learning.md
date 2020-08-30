---
title: 初识 Shell 脚本
date: 2020-08-23 10:00:45
tags: Shell脚本
categories: Shell
---

## 概述

Shell 是一种脚本语言，那么，就必须有解释器来执行这些脚本，常见的脚本解释器有：

- **bash**：是 Linux 标准默认的 shell。bash 由 Brian Fox 和 Chet Ramey 共同完成，是 BourneAgain Shell 的缩写，内部命令一共有 40 个。

- **sh** ：由 Steve Bourne 开发，是 Bourne Shell 的缩写，sh 是 Unix 标准默认的 shell。

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

Shell 经过了 POSIX 的标准化，使其可以在不同的 Linux 系统上进行移植，它属于解释性语言。本文关注的是 bash，也就是 Bourne Again Shell，由于易用和免费，Bash 在日常工作中被广泛使用。同时，Bash 也是大多数Linux 系统默认的 Shell。Shell 的运行模式大概分为两种：

- 交互式：在**终端**直接输入命令进行执行，并等待执行完毕后再去执行下一条命令。

- 非交互式：以非交互式方式执行的shell就是运行过程中不需要与用户输入输出打交道的shell。其实就是命令的集合，将命令集合直接放在一个脚本文件中，然后由 Bash Shell 读取并执行。一般情况下运维人员会把需要处理的命令逻辑写入脚本文件中，一次执行即可。也就是实现了 Linux 的自动化运维，效率也更高。

本文中讲解的 Shell 脚本都是基于非交互式，这样更容易对某一具体脚本功能做集中化管理。

## 第一个 Shell 脚本

按照旧例，我们先从一个 Hello world 开始，首先创建一个 `sh` 格式的文件 `hello_world.sh ` ：

```
touch hello_world.sh
```

接着打开该文件，并加入如下 code：

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

`chmod +x` 命令是为了让 hello_world.sh 拥有可执行权限，接着通过 `./hello_world.sh` 就可以成功执行我们的第一个脚本啦，可以看到终端的确打印出了预期的提示。**echo** 命令是 Shell 的一个内部指令，用于在屏幕上打印出指定的字符串。除此以外还有 `printf` 命令，可以实现更加强大的格式化输出操作，它们的用法将在后面介绍。

和大多数语言一样，Shell 也拥有完整的语法结构，例如变量、函数、表达式、运算符、语句块等等，其中很多规则和主流语言类似，但作为脚本语言，它也拥有一些特殊的技能，我们下面一起来了解一下。

## Shell 中的变量

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

### 字符串

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

### 数组

Shell 中同样拥有数组类型，bash 支持一维数组（不支持多维数组），并且没有限定数组的大小。与其他主流语言类似，数组元素的下标由 0 开始编号，获取数组中的元素要利用下标，下标可以是整数或算术表达式，其值应大于或等于 0。我们可以通过如下方式来定义和使用数组：

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

### 变量类型

运行 shell 时，会同时存在如下三种变量：

- **局部变量** 

  局部变量在脚本或命令中定义，仅在当前 shell 实例中有效，其他 shell 启动的程序不能访问局部变量。

- **环境变量** 

  所有的程序，包括 shell 启动的程序，都能访问环境变量，有些程序需要环境变量来保证其正常运行。必要的时候 shell 脚本也可以定义环境变量。

- **shell 变量** 

  shell 变量是由 shell 程序设置的特殊变量。shell 变量中有一部分是环境变量，有一部分是局部变量，这些变量保证了 shell 的正常运行

本文之前的示例中一直都在使用局部变量，环境变量部分涉及到内容还比较多，本文篇幅有限，后续会专门针对环境变量来发文一篇，详细介绍环境变量的分类以及如何查看和配置。

## Shell 的参数传递

如果参数只能控制在脚本代码内部，那么它就失去了灵活性，因此，Shell 提供了参数传递功能，可以让我们在执行脚本时传入参数，就像给一个函数传参一样。我们先来看下面这个简单示例：

```shell
#!/bin/bash
echo "输入的玩家昵称: $1"
echo "输入的玩家分数: $2"
echo "玩家信息的种类: $#"
echo "玩家信息: $*"
```

通过以下命令格式来执行：

```
➜  Desktop ./hello_world.sh Jack 100
输入的玩家昵称: Jack
输入的玩家分数: 100
玩家信息的种类: 2
玩家信息: Jack 100
```

我们通过 `$1...$n` 来表示用户输入的第几个参数变量，可以看到我们在终端输入了两个参数，`$1` 代表我们输入的第一个参数 `Jack`，而 `$2` 则代表输入的第二个参数值 `100`。除此以外，Shell 还提供了其他特殊字符来处理参数：

| 参数格式 | 说明                                                         |
| :------- | :----------------------------------------------------------- |
| $#       | 传递到脚本的参数个数                                         |
| $*       | 以一个单字符串显示所有向脚本传递的参数。 如"$*"用 `"` 括起来的情况、以"$1 $2 … $n"的形式输出所有参数。 |
| $$       | 脚本运行的当前进程ID号                                       |
| $!       | 后台运行的最后一个进程的ID号                                 |
| $@       | 与$*相同，但是使用时加引号，并在引号中返回每个参数。 如"$@"用 `"` 括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。 |
| $-       | 显示 Shell 当前使用的选项，与 set 命令功能相同。             |
| $?       | 显示最后命令的退出状态。0 表示没有错误，其他任何值表明有错误。 |

## Shell 中的运算符

与其他编程语言类似，Shell 中也提供了很多丰富的运算符，下面我们来一起认识一下。

### 算数运算符

原生 bash 虽然没有提供算数运算功能，但可以通过其他命令来实现，常见的有 `awk` 和 `expr`，一般我们都使用 `expr` 就可以。举个例子，我们可以通过下面语句实现加法运算：

```shell
#!/bin/bash
math_score=94
english_score=90
sum_score=`expr $math_score + $english_score`
echo "学生数学成绩: ${math_score}"
echo "学生英语成绩: ${english_score}"
echo "学生总分为: ${sum_score}"
```

我们需要注意两点：

1. 运算符和表达式之间要留有空格，否则会出错
2. 运算表达式需要被反引号 ``` `` 包裹

下面列举一下常见的算数运算符，并给出用法示例：

| 运算符 | 说明                                          | 举例（a=5, b=4）             |
| :----- | :-------------------------------------------- | :--------------------------- |
| +      | 加法                                          | `expr $a + $b` 结果为 9。    |
| -      | 减法                                          | `expr $a - $b` 结果为 1。    |
| *      | 乘法                                          | `expr $a \* $b` 结果为  20。 |
| /      | 除法                                          | `expr $b / $a` 结果为 1。    |
| %      | 取余                                          | `expr $b % $a` 结果为 4。    |
| =      | 赋值                                          | a=$b 将把变量 b 的值赋给 a。 |
| ==     | 相等。用于比较两个数字，相同则返回 true。     | [ $a == $b ] 返回 false。    |
| !=     | 不相等。用于比较两个数字，不相同则返回 true。 | [ $a != $b ] 返回 true。     |

### 关系运算符

关系运算符只支持数字，不支持字符串，除非字符串的值是数字。下表列出了常用的关系运算符，假设变量 `a` 为 6，变量 `b` 为 8：

| 运算符 | 说明                                                  | 举例                       |
| :----- | :---------------------------------------------------- | :------------------------- |
| -eq    | 检测两个数是否相等，相等返回 true。                   | [ $a -eq $b ] 返回 false。 |
| -ne    | 检测两个数是否不相等，不相等返回 true。               | [ $a -ne $b ] 返回 true。  |
| -gt    | 检测左边的数是否大于右边的，如果是，则返回 true。     | [ $a -gt $b ] 返回 false。 |
| -lt    | 检测左边的数是否小于右边的，如果是，则返回 true。     | [ $a -lt $b ] 返回 true。  |
| -ge    | 检测左边的数是否大于等于右边的，如果是，则返回 true。 | [ $a -ge $b ] 返回 false。 |
| -le    | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | [ $a -le $b ] 返回 true。  |

### 布尔运算符

下表列出了常用的布尔运算符，假定变量 `a` 为 1，变量 `b` 为 2：

| 运算符 | 说明                                                | 举例                                   |
| :----- | :-------------------------------------------------- | :------------------------------------- |
| !      | 非运算，表达式为 true 则返回 false，否则返回 true。 | [ ! false ] 返回 true。                |
| -o     | 或运算，有一个表达式为 true 则返回 true。           | [ $a -lt 2 -o $b -gt 10 ] 返回 true。  |
| -a     | 与运算，两个表达式都为 true 才返回 true。           | [ $a -lt 2 -a $b -gt 10 ] 返回 false。 |

### 逻辑运算符

下面看一下 Shell 的逻辑运算符，我们假设变量 a 为 1，变量 b 为 2:

| 运算符 | 说明       | 举例                                   |
| :----- | :--------- | :------------------------------------- |
| &&     | 逻辑的 AND | [[ $a -lt 6 && $b -gt 6 ]] 返回 false  |
| \|\|   | 逻辑的 OR  | [[ $a -lt 6 \|\| $b -gt 6 ]] 返回 true |

### 字符串运算符

下表列出了常用的字符串运算符，我们假设变量 `a` 为 "Jack"，变量 `b` 为 "Tom"：

| 运算符 | 说明                                         | 举例                     |
| :----- | :------------------------------------------- | :----------------------- |
| =      | 检测两个字符串是否相等，相等返回 true。      | [ $a = $b ] 返回 false。 |
| !=     | 检测两个字符串是否相等，不相等返回 true。    | [ $a != $b ] 返回 true。 |
| -z     | 检测字符串长度是否为0，为0返回 true。        | [ -z $a ] 返回 false。   |
| -n     | 检测字符串长度是否不为 0，不为 0 返回 true。 | [ -n "$a" ] 返回 true。  |
| $      | 检测字符串是否为空，不为空返回 true。        | [ $a ] 返回 true。       |

### 文件测试运算符

文件测试运算符用于检测 Unix 文件的各种属性。常见的属性检测能力如下：

| 操作符  | 说明                                                         |
| :------ | :----------------------------------------------------------- |
| -b file | 检测文件是否是块设备文件，如果是，则返回 true。              |
| -c file | 检测文件是否是字符设备文件，如果是，则返回 true。            |
| -d file | 检测文件是否是目录，如果是，则返回 true。                    |
| -f file | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 |
| -g file | 检测文件是否设置了 SGID 位，如果是，则返回 true。            |
| -k file | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。  |
| -p file | 检测文件是否是有名管道，如果是，则返回 true。                |
| -u file | 检测文件是否设置了 SUID 位，如果是，则返回 true。            |
| -r file | 检测文件是否可读，如果是，则返回 true。                      |

值得一提的是，Shell 里面的**中括号** ( 即 `[]` ) 可用于一些条件的测试，上述的运算符一节已经看到它出现了很多次，可以用来做算术比较、字符串比较、文件属性的逻辑判断等等，后续我们编写脚本过程中会经常和它打交道。判断条件如果太多可能容易出错，这时候我们可以通过 test 命令来代替执行条件检测，如：

```shell
if [ $var -eq 0 ]; then echo "True"; fi
# 上下等价
if test $var -eq 0; then echo "True"; fi
```

流程控制语句作为编程语言中至关重要的组成部分，Shell 自然也具备丰富的流程控制语句。思想都是殊途同归，只不过写法上有所差异，需要我们多练习和熟悉。下面我们着重介绍一下其中的条件语句和循环语句。

## Shell 中的条件语句

### if - else 语句

bash 中的 **if-else** 语句写法比较特别，我们先从下面这个简单的例子看起：

```shell
#!/bin/bash
math_score=94
english_score=90
sum_score=`expr $math_score + $english_score`
# if else
if [ $math_score == 100 ]; then
	echo "perfect score!"
elif [ $math_score -gt 90] && [ $math_score -lt 100 ]; then
	echo "good job!"
else
	echo "normal score"
fi
```

可以看到，我们需要在 if 语句中额外添加 `then` 关键字，接着另起一行执行当前条件下的逻辑；另外，`elif` 关键字就比较好理解了，即常见的 `else-if`，需要注意的是，我们需要在 if-else 语句最后添加限定符 `fi` 来表示当前条件语句的结束。此外，上述的脚本程序也可以写成如下格式：

```shell
#!/bin/bash
math_score=94
english_score=90
sum_score=`expr $math_score + $english_score`
# if else
if [ $math_score == 100 ] 
then
	echo "perfect score!"
elif [ $math_score -gt 90] && [ $math_score -lt 100 ]
then
	echo "good job!"
else
	echo "normal score"
fi
```

### case 语句

Shell 中的 case 语句可以简单理解为其他语言中的 `switch-case` 语句，即多选择语句，适用于选择场景较多的情况。下面，我们先看一个简单示例：

```shell
#!/bin/bash
echo '请输入你的数学考试成绩:'
read score
case $score in
	99|100)	echo '太棒了！'
	;;
	9[0-8])  echo '很优秀！'
	;;
	[6-8][0-9])  echo '表现的还可以！'
	;;
	*)  echo '下次要努力！'
	;;
esac
```

从上述示例可以看到，我们 case 语句与其他语言中的写法也不太一样，这里可以做出以下简单总结：

- **`case $参数 in`** 是标准写法格式，每一个条件后需要以 **`)`** 右括号来跟随，固定格式，可以联想记忆为 Java 当中的冒号。
- 我们可以使用固定值来作为匹配项，也可以使用 **`|`** 来分隔多个需要执行相同逻辑的条件，或者用**正则表达式**来匹配。
- 每一个条件逻辑处理完要换行紧跟 **`;;`**，可以理解为 Java 当中的 `break`。
- 至于最后的 **`*)`** 格式类似于 Java 中的 `default` 关键字功能。 
- 最终也需要添加 **`esac`** 关键字来表示语句的结束。
- 顺带一提，这里开头的 `read` 命令用来获取终端用户的输入内容，并传递给 `score` 变量。

## Shell 中的循环语句

Shell 中同样支持各种循环操作，常见的有for、while、until 等，下面我们来具体看一下它们的用法。

### for 循环

Shell 中 for 循环比较简单，示例如下：

```shell
#!/bin/bash
score_array=(89 78 99 72 100)
for score in ${score_array[@]}
do
	echo "学生的成绩依次是: ${score}"
done
#or
for score in 89 78 99 72 100
do
	echo "学生的成绩依次是: ${score}"
done
```

可以看到上面是两种方式的 for 循环，比较简单，就不细说了。

### while 循环

关于 while 循环的示例如下：

```shell
#!/bin/bash
number=1
while [ $number -le 8 ]
do
	echo "当前数字: ${number}"
	let "number++"
done
```

上述程序用来输出小于等于 8 的数字，输出结果如下：

```
当前数字: 1
当前数字: 2
当前数字: 3
当前数字: 4
当前数字: 5
当前数字: 6
当前数字: 7
当前数字: 8
```

**let** 命令用于执行表达式，表达式中不需要 `$` 符号来修饰变量。当然，提到 while 就不得不说一下比较常用的无限循环了，其实很简单：

```shell
#!/bin/bash
while :
do
	echo '德玛西亚，永世长存'
done
#or
while true; do
	echo '德玛西亚，永世长存'
done
```

### until 循环

until 与 while 正好相反，它是在条件为 false 时继续执行，直到 true 时停止循环，我们将上面的用 while 循环实现的输出 1-8 数字的例子用 until 实现一下：

```shell
#!/bin/bash
number=1
until [[ $number -gt 8 ]]; do
	echo "当前数字: ${number}"
	let "number++"
done
```

## Shell 中的函数

函数与命令的执行结果可以作为条件语句使用。要注意的是，和 C 语言不同，shell 语言中 0 代表 true，0 以外的值代表 false。针对函数，我们举个简单的示例：

```shell
#!/bin/bash
function calculateTotalScore() {
	total_score=0
	score_chinese=$1
	score_math=$2
	score_english=$3
	total_score=`expr ${score_chinese} + ${score_math} + ${score_english}`
	echo "这位学生的总分为: $total_score"
}

calculateTotalScore 92 100 89
#执行结果：这位学生的总分为: 281
```

函数的传参我们通过 **$n** 来获取，最终执行的时候只需要在函数后追加参数即可。

## 其他常见命令

`echo` 命令先前使用的时候已经讲过了，下面我们来看看与它类似的命令 `printf`。

### printf 命令

`printf` 命令使用引用文本或空格分隔的参数，外面可以在 printf 中使用格式化字符串，还可以定制字符串的宽度、左右对齐方式等。我们举个简单的示例：

```shell
#!/bin/bash
echo "学生数学成绩表："
printf "%-6s %-6d\n" '小明' 98
printf "%-6s %-6d\n" '小红' 89
```

执行结果如下：

```
➜  Desktop ./hello_world.sh 
学生数学成绩表：
小明 98    
小红 89 
```

这里的 `%s`、`%d` 作为格式替换符，可以作为占位符，根据后续传入的具体值来替换对应位置的符号。上述示例 `%-6s` 中的 `-6` 表示表示左对齐，没有 `-` 符合则表示右对齐，任何字符都会被显示在 6 个字符宽的字符内，如果不足则自动以空格填充，超过也会将内容全部显示出来。下面来看下常见的格式替换符以及其含义：

| 格式替换符 | 含义             | 描述                                         |
| ---------- | ---------------- | -------------------------------------------- |
| %s         | 字符串替换符     | 对应位置参数必须是字符串或者字符型，否则报错 |
| %d         | 十进制整数替换符 | 对应位置参数必须是十进制整数，否则报错       |
| %c         | 字符替换符       | 对应位置参数必须是字符串或者字符型，否则报错 |
| %f         | 浮点数替换符     | 对应位置参数必须是浮点数或整数，否则报错     |

具体的效果可以自己尝试在脚本中编写看下效果。此外，printf 命令中还支持转义字符，如示例中的 `/n` 表示换行符，成功在终端生效了。如果没有指定具体参数的值，那么格式替换符会被数据类型下的默认值代替，如 `%s` 用 NULL 代替，`%d` 用 0 代替。

### test 命令

Shell 中的 test 命令用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试。我们直接看下面示例：

```shell
#!/bin/bash
score_math=99
score_english=68
if test $[score_math] -gt $[score_english]
then
	echo 'Good at math'
else
	echo 'Good at english'
fi

read input_name
target_name="Tom"
if test $input_name = $target_name
then
    echo 'Yes, is Tom!'
else
    echo 'No, not him!'
fi


cd /Destop
if test -e ./hello_world.sh -o -e ./Frame.png
then
    echo '至少有一个文件存在!'
else
    echo '两个文件都不存在'
fi
```

值得一提的是，除了使用 `expr` 来 执行算数运算，我们也可以在 `[]` 来实现算数运算，如：

```shell
#!/bin/bash
a=10
b=8
c=3
result=$[a+b*c]
echo "计算结果为: $result"
#计算结果为:34
```

本文关于 Shell 脚本的基础介绍就到这里，篇幅比较长，只涉及到 Shell 中的基础语法和用法，后续会涉及到具体应用实战和高级用法补充。



 