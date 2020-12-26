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

## 变量替换，命令替换，转义字符

### 转义字符

























###