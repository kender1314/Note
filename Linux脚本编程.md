# Linux脚本编程

## Shell命令的本质

​	真正能够控制计算机硬件（CPU、内存、显示器等）的只有操作系统内核（Kernel），图形界面和命令行只是架设在用户和内核之间的一座桥梁。

​	由于安全、复杂、繁琐等原因，用户不能直接接触内核（也没有必要），需要另外再开发一个程序，让用户直接使用这个程序；该程序的作用就是接收用户的操作（点击图标、输入命令），并进行简单的处理，然后再传递给内核，这样用户就能间接地使用操作系统内核了。

​	用户界面和命令行就是这个另外开发的程序，就是这层“代理”。在Linux下，这个命令行程序叫做 **Shell**。

​	Shell 是一个应用程序，它连接了用户和 Linux 内核，让用户能够更加高效、安全、低成本地使用 Linux 内核，这就是 Shell 的本质。

​	Shell 本身并不是内核的一部分，它只是站在内核的基础上编写的一个应用程序，它和 QQ、迅雷、Firefox 等其它软件没有什么区别。然而 Shell 也有着它的特殊性，就是开机立马启动，并呈现在用户面前；用户通过 Shell 来使用 Linux，不启动 Shell 的话，用户就没办法使用 Linux。

​	操作系统内核（Kernel）暴露出来的是一个一个函数，只用调用函数，才能使用内核，没有其他途径。

​	Shell 本身支持的命令并不多，功能也有限，但是 Shell 可以调用其他的程序，每个程序就是一个命令，这使得 Shell 命令的数量可以无限扩展，其结果就是 Shell 的功能非常强大，完全能够胜任 Linux 的日常管理工作，如文本或字符串检索、文件的查找或创建、大规模软件的自动部署、更改系统设置、监控服务器性能、发送报警邮件、抓取网页内容、压缩文件等。

​	可以将 Shell 在整个 Linux 系统中的地位描述成下图所示的样子。注意“用户”和“其它应用程序”是通过虚线连接的，因为用户启动 Linux 后直接面对的是 Shell，通过 Shell 才能运行其它的应用程序。

![Shell在整个Linux系统中的地位示意图](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201226110844.gif)

​	现代 Linux 系统在启动时会自动创建几个虚拟控制台（Virtual Console），其中一个供图形桌面程序使用，其他的保留原生控制台的样子。虚拟控制台其实就是 Linux 系统内存中运行的虚拟终端（Virtual Terminal）。



## Shell 是一种脚本语言

​	任何代码最终都要被“翻译”成二进制的形式才能在计算机中执行。

​	有的编程语言，如 C/C++、Pascal、Go语言、汇编等，必须在程序运行之前将所有代码都翻译成二进制形式，也就是生成可执行文件，用户拿到的是最终生成的可执行文件，看不到源码。

​	**这个过程叫做编译（Compile），这样的编程语言叫做编译型语言，完成编译过程的软件叫做编译器（Compiler）。**

​	而有的编程语言，如 Shell、[JavaScript](http://c.biancheng.net/js/)、Python、[PHP](http://c.biancheng.net/php/)等，需要一边执行一边翻译，不会生成任何可执行文件，用户必须拿到源码才能运行程序。程序运行后会即时翻译，翻译完一部分执行一部分，不用等到所有代码都翻译完。

​	**这个过程叫做解释，这样的编程语言叫做解释型语言或者脚本语言（Script），完成解释过程的软件叫做解释器。**

​	编译型语言的优点是执行速度快、对硬件要求低、保密性好，适合开发操作系统、大型应用程序、数据库等。

​	脚本语言的优点是使用灵活、部署容易、跨平台性好，非常适合 Web 开发以及小工具的制作。

​	Shell 就是一种脚本语言，我们编写完源码后不用编译，直接运行源码即可。

## Shell有哪些

### 几种常见的Shell：sh、bash、csh、tcsh、ash

几种常见的Shell：sh、bash、csh、tcsh、ash

#### sh

sh 的全称是 Bourne shell，由 AT&T 公司的 Steve Bourne开发，为了纪念他，就用他的名字命名了。

sh 是 UNIX 上的标准 shell，很多 UNIX 版本都配有 sh。sh 是第一个流行的 Shell。

#### csh

sh 之后另一个广为流传的 shell 是由柏克莱大学的 Bill Joy 设计的，这个 shell 的语法有点类似C语言，所以才得名为 C shell ，简称为 csh。

Bill Joy 是一个风云人物，他创立了 BSD 操作系统，开发了 vi 编辑器，还是 Sun 公司的创始人之一。

> BSD 是 UNIX 的一个重要分支，后人在此基础上发展出了很多现代的操作系统，最著名的有 FreeBSD、OpenBSD 和 NetBSD，就连 Mac OS X 在很大程度上也基于BSD。

#### tcsh

tcsh 是 csh 的增强版，加入了命令补全功能，提供了更加强大的语法支持。

#### ash

一个简单的轻量级的 Shell，占用资源少，适合运行于低内存环境，但是与下面讲到的 bash shell 完全兼容。

#### <font style="color: red">bash</font>

bash shell 是 Linux 的默认 shell，本教程也基于 bash 编写。

bash 由 GNU 组织开发，保持了对 sh shell 的兼容性，是各种 Linux 发行版默认配置的 shell。

> bash 兼容 sh 意味着，针对 sh 编写的 Shell 代码可以不加修改地在 bash 中运行。

尽管如此，bash 和 sh 还是有一些不同之处：

- 一方面，bash 扩展了一些命令和参数；
- 另一方面，bash 并不完全和 sh 兼容，它们有些行为并不一致，但在大多数企业运维的情况下区别不大，特殊场景可以使用 bash 代替 sh。

### 查看 Shell

Shell 是一个程序，一般都是放在`/bin`或者`/user/bin`目录下，当前 Linux 系统可用的 Shell 都记录在`/etc/shells`文件中。`/etc/shells`是一个纯文本文件，你可以在图形界面下打开它，也可以使用 cat 命令查看它。

通过 cat 命令来查看当前 Linux 系统的可用 Shell：

```
$ cat /etc/shells
```

![image-20201226113057505](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201226113058.png)

查看当前 Linux 的默认 Shell：

```
echo $SHELL
```

![image-20201226113331930](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201226113332.png)



## 第一个脚本

脚本的扩展名为sh（sh即代表shell）。

新建first-shell.sh文件，输入以下内容：

```
#!/bin/bash
echo "Hello World !"
```

“#!” 是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行，即使用哪一种Shell。echo命令用于向窗口输出文本。

**作为可执行程序**

将上面的代码保存为test.sh，并 cd 到相应目录：

```
chmod +x ./test.sh  #使脚本具有执行权限
./test.sh  #执行脚本
```

注意，一定要写成./test.sh，而不是test.sh。运行其它二进制的程序也一样，直接写test.sh，linux系统会去PATH里寻找有没有叫test.sh的，而只有/bin, /sbin, /usr/bin，/usr/sbin等在PATH里，你的当前目录通常不在PATH里，所以写成test.sh是会找不到命令的，要用./test.sh告诉系统说，就在当前目录找。

通过这种方式运行bash脚本，第一行一定要写对，好让shell查找到正确的解释器。如果是使用/bin/sh作为解释器的脚本，那么就可以省略第一行。

**脚本作为解释器参数**

这种运行方式是，直接运行解释器，其参数就是shell脚本的文件名，如：

```
/bin/sh test.sh
/bin/php test.php
```

这种方式运行的脚本，不需要在第一行指定解释器信息，写了也没用。

再看一个例子。下面的脚本使用 **read** 命令从 stdin 获取输入并赋值给 PERSON 变量，最后在 stdout 上输出：

```
#!/bin/bash# Author : mozhiyan# Copyright (c) http://see.xidian.edu.cn/cpp/linux/# Script follows here:echo "What is your name?"read PERSONecho "Hello, $PERSON"
```

运行脚本：

```
chmod +x ./test.sh
$./test.sh
What is your name?
mozhiyan
Hello, mozhiyan
$
```

## Shell变量

在 Bash shell 中，每一个变量的值都是字符串，无论你给变量赋值时有没有使用引号，值都会以字符串的形式存储。这意味着，Bash shell 在默认情况下不会区分变量类型，即使你将整数和小数赋值给变量，它们也会被视为字符串。

当然，如果有必要，你也可以使用 declare 关键字显式定义变量的类型。

Shell 变量的命名规范和大部分编程语言都一样：

- [x] 变量名由数字、字母、下划线组成；
- [x] 必须以字母或者下划线开头；
- [x] 不能使用 Shell 里的关键字（通过 help 命令可以查看保留关键字）。

### 变量类型

运行shell时，会同时存在三种变量：

**1) 局部变量**

局部变量在脚本或命令中定义，仅在当前shell实例中有效，其他shell启动的程序不能访问局部变量。

**2) 环境变量**

所有的程序，包括shell启动的程序，都能访问环境变量，有些程序需要环境变量来保证其正常运行。必要的时候shell脚本也可以定义环境变量。

**3) shell变量**

shell变量是由shell程序设置的特殊变量。shell变量中有一部分是环境变量，有一部分是局部变量，这些变量保证了shell的正常运行

### 定义变量

Shell 支持以下三种定义变量的方式：

```
variable=value
variable='value'
variable="value"
```

variable 是变量名，value 是赋给变量的值。如果 value 不包含任何空白符（例如空格、Tab缩进等），那么可以不使用引号；如果 value 包含了空白符，那么就必须使用引号包围起来。使用单引号和使用双引号也是有区别的。

- [x] 注意，赋值号的周围不能有空格，这可能和你熟悉的大部分编程语言都不一样。

### 单引号和双引号的区别

新建quotation.sh：

```
#!/bin/bash
url="www.baidu.com"
website1='这是网站链接：${url}'
website2="这是网站链接：${url}"
echo ${website1}
echo ${website2}
```

website1使用的是单引号，website2使用的是双引号。运行结果如下：

![image-20201226140137006](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201226140139.png)

可以看出，单引号是不会打印变量的值的，适用于不希望解析变量、命令等的场景。双引号可以打印出变量的值。

如果变量的内容是数字，那么可以不加引号；如果真的需要原样输出就加单引号；其他没有特别要求的字符串等最好都加上双引号，定义变量时加双引号是最常见的使用场景。

### 使用变量

使用一个定义过的变量，只要在变量名前面加美元符号`$`即可，如：

```
echo $authorecho ${author}
```

变量名外面的花括号`{ }`是可选的，加不加都行，加花括号是为了帮助解释器识别变量的边界，比如下面这种情况：

```
skill="Java"echo "I am good at ${skill}Script"
```

如果不给 skill 变量加花括号，写成`echo "I am good at $skillScript"`，解释器就会把 $skillScript 当成一个变量（其值为空），代码执行结果就不是我们期望的样子了。

- [x] 推荐给所有变量加上花括号`{ }`，这是个良好的编程习惯。

### 只读变量

使用 **readonly** 命令可以将变量定义为只读变量，只读变量的值不能被改变。

下面的例子尝试更改只读变量，结果报错：

```
#!/bin/bash
name="jiang"
echo "Hello World! ${name}"
readonly name
name="wu"
echo "${name}"
```

执行结果：

![image-20201226141458065](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201226141459.png)

### 删除变量

使用 **unset** 命令可以删除变量。语法：

```
unset variable_name
```

变量被删除后不能再次使用；unset 命令不能删除只读变量。

举个例子：

```
#!/bin/sh
myUrl="http://see.xidian.edu.cn/cpp/u/xitong/"
unset myUrl
echo $myUrl
```

上面的脚本没有任何输出。

## 将命令的结果赋值给变量

Shell 也支持将命令的执行结果赋值给变量，常见的有以下两种方式：

```
variable=`command`
variable=$(command)
```

第一种方式把命令用反引号包围起来，反引号和单引号非常相似，容易产生混淆，所以不推荐使用这种方式；第二种方式把命令用`$()`包围起来，区分更加明显，所以推荐使用这种方式。

![image-20201226141152064](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201226141153.png)

这个也可以写在脚本文件中，创建date-test.sh：

```
#!/bin/bash
DATE=$(date)
CAT=`cat first-shell.sh`
echo "${DATE}"
echo "${CAT}"
```

执行命令**./date-test.sh**查看结果

![image-20201226151200340](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201226151201.png)

## Shell特殊变量

<center><b>特殊变量列表</b></center>

| 变量 | 含义                                                         |
| ---- | ------------------------------------------------------------ |
| $0   | 当前脚本的文件名                                             |
| $n   | 传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。 |
| $#   | 传递给脚本或函数的参数个数。                                 |
| $*   | 传递给脚本或函数的所有参数。                                 |
| $@   | 传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同，下面将会讲到。 |
| $?   | 上个命令的退出状态，或函数的返回值。                         |
| $$   | 当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。 |

### 例子

新建input.sh文件，输入内容：

```
#!/bin/bash
echo "file name: $0"
echo "first parameter: $1"
echo "second paramter: $2"
echo "parameter cout: $#"
echo "all parameter: $*"
echo "all parameter: $@"
echo "$?"
echo "current process ID: $$"
```

执行下面的下面的命令：

```
./input.sh 23 55
```

执行结果：

![image-20201226144122310](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201226144123.png)



### $* 和 $@ 的区别

$* 和 $@ 都表示传递给函数或脚本的所有参数，不被双引号(" ")包含时，都以"$1" "$2" … "$n" 的形式输出所有参数。

但是当它们被双引号(" ")包含时，"$*" 会将所有的参数作为一个整体，以"$1 $2 … $n"的形式输出所有参数；"$@" 会将各个参数分开，以"$1" "$2" … "$n" 的形式输出所有参数。

下面的例子可以清楚的看到 $* 和 $@ 的区别，建立input1.sh文件：

```
#!/bin/bash
echo "\$*=" $*
echo "\"\$*\"=" "$*"
echo "\$@=" $@
echo "\"\$@\"=" "$@"
echo "print each param from \$*"
for var in $*
do
    echo "$var"
done
echo "print each param from \$@"
for var in $@
do
    echo "$var"
done
echo "print each param from \"\$*\""
for var in "$*"
do
    echo "$var"
done
echo "print each param from \"\$@\""
for var in "$@"
do
    echo "$var"
done
```

运行结果：

![image-20201226145247376](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201226145248.png)

### $?退出状态

$? 可以获取上一个命令的退出状态。所谓退出状态，就是上一个命令执行后的返回结果。退出状态是一个数字，一般情况下，大部分命令执行成功会返回 0，失败返回 1。不过，也有一些命令返回其他值，表示不同类型的错误。

$? 也可以表示函数的返回值。

## 变量替换，转义字符

### 转义字符

下面的转义字符都可以用在 echo 中：

| 转义字符 | 含义                             |
| -------- | -------------------------------- |
| \\       | 反斜杠                           |
| \a       | 警报，响铃                       |
| \b       | 退格（删除键）                   |
| \f       | 换页(FF)，将当前位置移到下页开头 |
| \n       | 换行                             |
| \r       | 回车                             |
| \t       | 水平制表符（tab键）              |
| \v       | 垂直制表符                       |

举个例子：

```
#!/bin/basha=10echo -e "Value of a is $a \n"
```

运行结果：

```
Value of a is 10
```

这里 -e 表示对转义字符进行替换。如果不使用 -e 选项，将会原样输出：

```
Value of a is 10\n
```

### 变量替换

变量替换可以根据变量的状态（是否为空、是否定义等）来改变它的值

可以使用的变量替换形式：

| 形式            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| ${var}          | 变量本来的值                                                 |
| ${var:-word}    | 如果变量 var 为空或已被删除(unset)，那么返回 word，但不改变 var 的值。 |
| ${var:=word}    | 如果变量 var 为空或已被删除(unset)，那么返回 word，并将 var 的值设置为 word。 |
| ${var:?message} | 如果变量 var 为空或已被删除(unset)，那么将消息 message 送到标准错误输出，可以用来检测变量 var 是否可以被正常赋值。 若此替换出现在Shell脚本中，那么脚本将停止运行。 |
| ${var:+word}    | 如果变量 var 被定义，那么返回 word，但不改变 var 的值。      |

示例：创建var-replace.sh

```
#!/bin/bash
var="hello world"
echo "print message1:${var}"
echo "print message2:${var:+word}"
echo "delete var"
unset var
echo "print message3:${var:-word}"
echo "print message4:${var:=word}"
echo "替换之后：${var}"
unset var
echo "打印错误：${var:?message}"
```

打印结果：

![image-20201226214425758](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/image-20201226214425758.png)

## Shell运算符



Bash 支持很多运算符，包括算数运算符、关系运算符、布尔运算符、字符串运算符和文件测试运算符。

### 算数运算符

原生bash不支持简单的数学运算，但是可以通过其他命令来实现，例如 awk 和 expr，expr 最常用。

注意：

- [x] 表达式和运算符之间要有空格，例如 2+2 是不对的，必须写成 2 + 2，这与我们熟悉的大多数编程语言不一样。
- [x] 完整的表达式要被 ``包含，注意这个字符不是常用的单引号，在 Esc 键下边。

- [x] 乘号(*)前边必须加反斜杠(\)才能实现乘法运算；

- [x] if后面[]与里面的条件之间，必须要有空格，否则无法识别

  <center><b>算术运算符列表</b></center>

| 运算符 | 说明                                          | 举例                          |
| ------ | --------------------------------------------- | ----------------------------- |
| +      | 加法                                          | `expr $a + $b` 结果为 30。    |
| -      | 减法                                          | `expr $a - $b` 结果为 10。    |
| *      | 乘法                                          | `expr $a \* $b` 结果为  200。 |
| /      | 除法                                          | `expr $b / $a` 结果为 2。     |
| %      | 取余                                          | `expr $b % $a` 结果为 0。     |
| =      | 赋值                                          | a=$b 将把变量 b 的值赋给 a。  |
| ==     | 相等。用于比较两个数字，相同则返回 true。     | [ $a == $b ] 返回 false。     |
| !=     | 不相等。用于比较两个数字，不相同则返回 true。 | [ $a != $b ] 返回 true。      |

举例：创建expr.sh

```
#!/bin/bash
a=20
b=10
val1=`expr $a + $b`
val2=`expr $a \* $b`
val3=`expr $a - $b`
val4=`expr $a / $b`
echo "the sum value is : ${val1}"
echo "the multiplication value is : ${val2}"
echo 'expr 8 - 20 = '${val3}
echo 'expr 20 /4 = '${val4}
if [ $a != $b ]
then
  echo "$a != $b"
fi
if [ $a == $b ]
then
  echo "$a == $b"
fi
```

运行结果：

![image-20201226220750090](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/image-20201226220750090.png)

### 关系运算符

关系运算符只支持数字，不支持字符串，除非字符串的值是数字。

<center><b>关系运算符列表</b></center>

| 运算符 | 说明                                                  | 举例                       |
| ------ | ----------------------------------------------------- | -------------------------- |
| -eq    | 检测两个数是否相等，相等返回 true。                   | [ $a -eq $b ] 返回 true。  |
| -ne    | 检测两个数是否相等，不相等返回 true。                 | [ $a -ne $b ] 返回 true。  |
| -gt    | 检测左边的数是否大于右边的，如果是，则返回 true。     | [ $a -gt $b ] 返回 false。 |
| -lt    | 检测左边的数是否小于右边的，如果是，则返回 true。     | [ $a -lt $b ] 返回 true。  |
| -ge    | 检测左边的数是否大等于右边的，如果是，则返回 true。   | [ $a -ge $b ] 返回 false。 |
| -le    | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | [ $a -le $b ] 返回 true。  |

关系运算符的例子：

```
#!/bin/sh

a=10
b=20
if [ $a -eq $b ]
then
   echo "$a -eq $b : a is equal to b"
else
   echo "$a -eq $b: a is not equal to b"
fi

if [ $a -ne $b ]
then
   echo "$a -ne $b: a is not equal to b"
else
   echo "$a -ne $b : a is equal to b"
fi

if [ $a -gt $b ]
then
   echo "$a -gt $b: a is greater than b"
else
   echo "$a -gt $b: a is not greater than b"
fi

if [ $a -lt $b ]
then
   echo "$a -lt $b: a is less than b"
else
   echo "$a -lt $b: a is not less than b"
fi

if [ $a -ge $b ]
then
   echo "$a -ge $b: a is greater or  equal to b"
else
   echo "$a -ge $b: a is not greater or equal to b"
fi

if [ $a -le $b ]
then
   echo "$a -le $b: a is less or  equal to b"
else
   echo "$a -le $b: a is not less or equal to b"
fi
```

运行结果：

```
10 -eq 20: a is not equal to b
10 -ne 20: a is not equal to b
10 -gt 20: a is not greater than b
10 -lt 20: a is less than b
10 -ge 20: a is not greater or equal to b
10 -le 20: a is less or  equal to b
```

### 布尔运算符

<center><b>布尔运算符列表</b></center>

| 运算符 | 说明                                                | 举例                                     |
| ------ | --------------------------------------------------- | ---------------------------------------- |
| !      | 非运算，表达式为 true 则返回 false，否则返回 true。 | [ ! false ] 返回 true。                  |
| -o     | 或运算，有一个表达式为 true 则返回 true。           | [ 10 -lt 20 -o 20 -gt 100 ] 返回 true。  |
| -a     | 与运算，两个表达式都为 true 才返回 true。           | [ 10 -lt 20 -a 20 -gt 100 ] 返回 false。 |

布尔运算符的例子：

```
#!/bin/sh
a=10
b=20
if [ $a != $b ]
then
   echo "$a != $b : a is not equal to b"
else
   echo "$a != $b: a is equal to b"
fi
if [ $a -lt 100 -a $b -gt 15 ]
then
   echo "$a -lt 100 -a $b -gt 15 : returns true"
else
   echo "$a -lt 100 -a $b -gt 15 : returns false"
fi
if [ $a -lt 100 -o $b -gt 100 ]
then
   echo "$a -lt 100 -o $b -gt 100 : returns true"
else
   echo "$a -lt 100 -o $b -gt 100 : returns false"
fi
if [ $a -lt 5 -o $b -gt 100 ]
then
   echo "$a -lt 100 -o $b -gt 100 : returns true"
else
   echo "$a -lt 100 -o $b -gt 100 : returns false"
fi
```

运行结果：

```
10 != 20 : a is not equal to b
10 -lt 100 -a 20 -gt 15 : returns true
10 -lt 100 -o 20 -gt 100 : returns true
10 -lt 5 -o 20 -gt 100 : returns false
```

### 字符串运算符

<center><b>字符串运算符列表</b></center>

| 运算符 | 说明                                      | 举例                     |
| ------ | ----------------------------------------- | ------------------------ |
| =      | 检测两个字符串是否相等，相等返回 true。   | [ $a = $b ] 返回 false。 |
| !=     | 检测两个字符串是否相等，不相等返回 true。 | [ $a != $b ] 返回 true。 |
| -z     | 检测字符串长度是否为0，为0返回 true。     | [ -z $a ] 返回 false。   |
| -n     | 检测字符串长度是否为0，不为0返回 true。   | [ -z $a ] 返回 true。    |
| str    | 检测字符串是否为空，不为空返回 true。     | [ $a ] 返回 true。       |

例子：创建string.sh

```
#!/bin/bash
a="heg"
b="jiang"
c="heg"
d=""
if [ $a = $c ]; then
  echo "a == c"
fi
if [ $a != $b ]; then
  echo "a != b"
fi
if [ -z$d ]; then
  echo "the size is 0 for d"
fi
if [ -n$a ]; then
  echo "the size is not 0 for a"
fi
if [ $a ]; then
  echo "a is not null"
fi
```

运行结果：

![image-20201226223044741](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/image-20201226223044741.png)

### 文件测试运算符

文件测试运算符用于检测 Unix 文件的各种属性。

例如，变量 file 表示文件“/var/www/tutorialspoint/unix/test.sh”，它的大小为100字节，具有 rwx 权限。下面的代码，将检测该文件的各种属性：

```
#!/bin/sh
file="/var/www/tutorialspoint/unix/test.sh"
if [ -r $file ]
then
   echo "File has read access"
else
   echo "File does not have read access"
fi
if [ -w $file ]
then
   echo "File has write permission"
else
   echo "File does not have write permission"
fi
if [ -x $file ]
then
   echo "File has execute permission"
else
   echo "File does not have execute permission"
fi
if [ -f $file ]
then
   echo "File is an ordinary file"
else
   echo "This is sepcial file"
fi
if [ -d $file ]
then
   echo "File is a directory"
else
   echo "This is not a directory"
fi
if [ -s $file ]
then
   echo "File size is zero"
else
   echo "File size is not zero"
fi
if [ -e $file ]
then
   echo "File exists"
else
   echo "File does not exist"
fi
```

运行结果：

```
File has read access
File has write permission
File has execute permission
File is an ordinary file
This is not a directory
File size is zero
File exists
```

<center><b>文件测试运算符列表</b></center>

|      |
| ---- |
|      |

| 操作符  | 说明                                                         | 举例                      |
| ------- | ------------------------------------------------------------ | ------------------------- |
| -b file | 检测文件是否是块设备文件，如果是，则返回 true。              | [ -b $file ] 返回 false。 |
| -c file | 检测文件是否是字符设备文件，如果是，则返回 true。            | [ -b $file ] 返回 false。 |
| -d file | 检测文件是否是目录，如果是，则返回 true。                    | [ -d $file ] 返回 false。 |
| -f file | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 | [ -f $file ] 返回 true。  |
| -g file | 检测文件是否设置了 SGID 位，如果是，则返回 true。            | [ -g $file ] 返回 false。 |
| -k file | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。  | [ -k $file ] 返回 false。 |
| -p file | 检测文件是否是具名管道，如果是，则返回 true。                | [ -p $file ] 返回 false。 |
| -u file | 检测文件是否设置了 SUID 位，如果是，则返回 true。            | [ -u $file ] 返回 false。 |
| -r file | 检测文件是否可读，如果是，则返回 true。                      | [ -r $file ] 返回 true。  |
| -w file | 检测文件是否可写，如果是，则返回 true。                      | [ -w $file ] 返回 true。  |
| -x file | 检测文件是否可执行，如果是，则返回 true。                    | [ -x $file ] 返回 true。  |
| -s file | 检测文件是否为空（文件大小是否大于0），不为空返回 true。     | [ -s $file ] 返回 true。  |
| -e file | 检测文件（包括目录）是否存在，如果是，则返回 true。          | [ -e $file ] 返回 true。  |

## Shell注释

以“#”开头的行就是注释，会被解释器忽略。

sh里没有多行注释，只能每一行加一个#号。只能像这样：

```
#--------------------------------------------
# 这是一个自动打ipa的脚本，基于webfrogs的ipa-build书写：
# https://github.com/webfrogs/xcode_shell/blob/master/ipa-build
# 功能：自动为etao ios app打包，产出物为14个渠道的ipa包
# 特色：全自动打包，不需要输入任何参数
#--------------------------------------------
##### 用户配置区 开始 #####
#
#
# 项目根目录，推荐将此脚本放在项目的根目录，这里就不用改了
# 应用名，确保和Xcode里Product下的target_name.app名字一致
#
##### 用户配置区 结束  #####
```

如果在开发过程中，遇到大段的代码需要临时注释起来，过一会儿又取消注释，怎么办呢？每一行加个#符号太费力了，可以把这一段要注释的代码用一对花括号括起来，定义成一个函数，没有地方调用这个函数，这块代码就不会执行，达到了和注释一样的效果。

## Shell字符串

字符串是shell编程中最常用最有用的数据类型（除了数字和字符串，也没啥其它类型好用了），字符串可以用单引号，也可以用双引号，也可以不用引号。单双引号的区别跟PHP类似。

### 单引号

```
str='this is a string'
```

单引号字符串的限制：

- 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的；
- 单引号字串中不能出现单引号（对单引号使用转义符后也不行）。

### 双引号

```
your_name='qinjx'str="Hello, I know your are \"$your_name\"! \n"
```

双引号的优点：

- 双引号里可以有变量
- 双引号里可以出现转义字符

### 拼接字符串

```
your_name="qinjx"greeting="hello, "$your_name" !"greeting_1="hello, ${your_name} !"echo $greeting $greeting_1
```

### 获取字符串长度

```
string="abcd"echo ${#string} #输出 4
```

### 提取子字符串

```
string="alibaba is a great company"echo ${string:1:4} #输出liba
```

### 查找子字符串

```
纯文本复制
string="alibaba is a great company"echo `expr index "$string" is`
```

## Shell数组

Shell在编程方面比Windows批处理强大很多，无论是在循环、运算。

<font style="color: red">bash支持一维数组（不支持多维数组），并且没有限定数组的大小。</font>类似与C语言，数组元素的下标由0开始编号。获取数组中的元素要利用下标，下标可以是整数或算术表达式，其值应大于或等于0。

### 定义数组

在Shell中，用括号来表示数组，数组元素用“空格”符号分割开。定义数组的一般形式为：
  array_name=(value1 ... valuen)
例如：

```
array_name=(value0 value1 value2 value3)
```

或者

```
array_name=(value0value1value2value3)
```


还可以单独定义数组的各个分量：

```
array_name[0]=value0array_name[1]=value1array_name[2]=value2
```

可以不使用连续的下标，而且下标的范围没有限制。

### 读取数组

读取数组元素值的一般格式是：
  ${array_name[index]}
例如：

```
valuen=${array_name[2]}
```

举个例子：

```
#!/bin/shNAME[0]="Zara"NAME[1]="Qadir"NAME[2]="Mahnaz"NAME[3]="Ayan"NAME[4]="Daisy"echo "First Index: ${NAME[0]}"echo "Second Index: ${NAME[1]}"
```

运行脚本，输出：

```
$./test.sh
First Index: Zara
Second Index: Qadir
```

使用@ 或 * 可以获取数组中的所有元素，例如：

```
${array_name[*]}${array_name[@]}
```

举个例子：

```
#!/bin/shNAME[0]="Zara"NAME[1]="Qadir"NAME[2]="Mahnaz"NAME[3]="Ayan"NAME[4]="Daisy"echo "First Method: ${NAME[*]}"echo "Second Method: ${NAME[@]}"
```

运行脚本，输出：

```
$./test.sh
First Method: Zara Qadir Mahnaz Ayan Daisy
Second Method: Zara Qadir Mahnaz Ayan Daisy
```

### 获取数组的长度

获取数组长度的方法与获取字符串长度的方法相同，例如：

```
纯文本复制
# 取得数组元素的个数length=${#array_name[@]}# 或者length=${#array_name[*]}# 取得数组单个元素的长度lengthn=${#array_name[n]}
```

## Shell echo命令

echo是Shell的一个内部指令，用于在屏幕上打印出指定的字符串。命令格式：

```
echo arg
```

您可以使用echo实现更复杂的输出格式控制。

### 显示转义字符

```
echo "\"It is a test\""
```

结果将是：
"It is a test"

双引号也可以省略。

### 显示变量

```
name="OK"echo "$name It is a test"
```

结果将是：
OK It is a test

同样双引号也可以省略。

如果变量与其它字符相连的话，需要使用大括号（{ }）：

```
mouth=8echo "${mouth}-1-2009"
```

结果将是：
8-1-2009

### 显示换行

```
echo "OK!\n"echo "It is a test"
```

输出：
OK!
It is a test

### 显示不换行

```
echo "OK!\c"echo "It is a test"
```

输出：
OK!It si a test

### 显示结果重定向至文件

```
echo "It is a test" > myfile
```

### 原样输出字符串

若需要原样输出字符串（不进行转义），请使用单引号。例如：

```
echo '$name\"'
```

### 显示命令执行结果

```
echo `date`
```

结果将显示当前日期

## shell printf命令

1. printf 命令用于格式化输出， 是echo命令的增强版。


*注意：printf 由 POSIX 标准所定义，移植性要比 echo 好。*

1. printf 不像 echo 那样会自动换行，必须显式添加换行符(\n)。


与C语言printf()函数的不同：

- printf 命令不用加括号
- format-string 可以没有引号，但最好加上，单引号双引号均可。
- 参数多于格式控制符(%)时，format-string 可以重用，可以将所有参数都转换。
- arguments 使用空格分隔，不用逗号。

例子：创建printf.sh

```
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
# 如果以 %d 的格式来显示字符串，那么会有警告，提示无效的数字，此时默认置为 0
printf "The first program always prints'%s,%d\n'" Hello Shell
```

运行结果：

![image-20210105195250473](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20210105195259.png)

## Shell if else语句

if 语句通过关系运算符判断表达式的真假来决定执行哪个分支。Shell 有三种 if ... else 语句：

- if ... fi 语句；
- if ... else ... fi 语句；
- if ... elif ... else ... fi 语句。

### if ... else 语句

if ... else 语句的语法：

```
if [ expression ]
then
   Statement(s) to be executed if expression is true
fi
```

如果 expression 返回 true，then 后边的语句将会被执行；如果返回 false，不会执行任何语句。

最后必须以 fi 来结尾闭合 if，fi 就是 if 倒过来拼写，后面也会遇见。

注意：expression 和方括号([ ])之间必须有空格，否则会有语法错误。

if ... else 语句也经常与 test 命令结合使用，如下所示：

```
num1=$[2*3]num2=$[1+5]if test $[num1] -eq $[num2]then    echo 'The two numbers are equal!'else    echo 'The two numbers are not equal!'fi
```

输出：

```
The two numbers are equal!
```

test 命令用于检查某个条件是否成立，与方括号([ ])类似。

## Shell test命令

Shell中的 test 命令用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试。

### **数值测试**

| 参数 | 说明           |
| ---- | -------------- |
| -eq  | 等于则为真     |
| -ne  | 不等于则为真   |
| -gt  | 大于则为真     |
| -ge  | 大于等于则为真 |
| -lt  | 小于则为真     |
| -le  | 小于等于则为真 |

例如：

```
num1=100
num2=100
if test $[num1] -eq $[num2]
then    
	echo 'The two numbers are equal!'
else    
	echo 'The two numbers are not equal!'
fi
```

输出：

```
The two numbers are equal!
```

### 字符串测试

| 参数      | 说明                 |
| --------- | -------------------- |
| =         | 等于则为真           |
| !=        | 不相等则为真         |
| -z 字符串 | 字符串长度伪则为真   |
| -n 字符串 | 字符串长度不伪则为真 |

例如：

```
num1=100
num2=100
if test num1=num2
then    
echo 
	'The two strings are equal!'
else    
	echo 'The two strings are not equal!'
fi
```

输出：

```
The two strings are equal!
```

### 文件测试

| 参数      | 说明                                 |
| --------- | ------------------------------------ |
| -e 文件名 | 如果文件存在则为真                   |
| -r 文件名 | 如果文件存在且可读则为真             |
| -w 文件名 | 如果文件存在且可写则为真             |
| -x 文件名 | 如果文件存在且可执行则为真           |
| -s 文件名 | 如果文件存在且至少有一个字符则为真   |
| -d 文件名 | 如果文件存在且为目录则为真           |
| -f 文件名 | 如果文件存在且为普通文件则为真       |
| -c 文件名 | 如果文件存在且为字符型特殊文件则为真 |
| -b 文件名 | 如果文件存在且为块特殊文件则为真     |

例如：

```
cd /bin
if test -e ./bash
then    
	echo 'The file already exists!'
else    
	echo 'The file does not exists!'
fi
```

输出：

```
The file already exists!
```

另外，Shell还提供了与( ! )、或( -o )、非( -a )三个逻辑操作符用于将测试条件连接起来，其优先级为：“!”最高，“-a”次之，“-o”最低。例如：

```
cd /bin
if test -e ./notFile -o ./bash
then    
	echo 'One file exists at least!'
else    
	echo 'Both dose not exists!'
fi
```

输出：

```
One file exists at least!
```

## Shell case esac语句

case ... esac 与其他语言中的 switch ... case 语句类似，是一种多分枝选择结构

<font style="color: red">case工作方式如上所示。取值后面必须为关键字 in，每一模式必须以右括号结束。取值可以为变量或常数。匹配发现取值符合某一模式后，其间所有命令开始执行直至 ;;。;; 与其他语言中的 break 类似，意思是跳到整个 case 语句的最后。</font>

<font style="color: red">取值将检测匹配的每一个模式。一旦模式匹配，则执行完匹配模式相应命令后不再继续其他模式。如果无一匹配模式，使用星号 * 捕获该值，再执行后面的命令。</font>

举例：创建case.sh

```
echo 'Input a number between 1 to 4'
echo 'Your number is:\c'
read num
case ${num} in
        1) echo 'You select 1'
        ;;
        2)  echo 'You select 2'
        ;;
        3)  echo 'You select 3'
        ;;
        4)  echo 'You select 4'
        ;;
        *)  echo 'You do not select a number between 1 to 4'
        ;;
esac
```

运行结果：

![image-20210105201811782](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20210105201817.png)

## Shell for循环

与其他编程语言类似，Shell支持for循环。

























###