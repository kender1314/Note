# Gradle笔记

## Gradle介绍

Gradle是一个基于Apache Ant和Apache Maven概念的项目自动化构建开源工具。它使用一种基于Groovy的特定领域语言(DSL)来声明项目设置，目前也增加了基于Kotlin语言的kotlin-based DSL，抛弃了基于XML的各种繁琐配置。

**领域特定语言**（英语：domain-specific language、DSL）指的是专注于某个[应用程序](https://baike.baidu.com/item/应用程序/5985445)领域的[计算机语言](https://baike.baidu.com/item/计算机语言/4456504)。，

## Groovy 介绍

Groovy 是 用于Java[虚拟机](https://baike.baidu.com/item/虚拟机)的一种敏捷的[动态语言](https://baike.baidu.com/item/动态语言)，它是一种成熟的[面向对象](https://baike.baidu.com/item/面向对象)编程语言，既可以用于面向对象编程，又可以用作纯粹的[脚本语言](https://baike.baidu.com/item/脚本语言)。使用该种语言不必编写过多的代码，同时又具有[闭包](https://baike.baidu.com/item/闭包)和动态语言中的其他特性。

### Java与Groovy比较

1. Groovy完全兼容Java语法，可以做脚本也可做类。
2. 分号是可选的，一般不加分号，以换行作为结束。
3. 类，方法，字段都是公开的，没有访问权限限制。
4. 默认生成具名（明值对）参数构造器key:value。
5. 字段不定义访问权限时，编译器自动给字段添加getter/setter方法。
6. 字段可使用点来获取，无访问权限的也可使用getter/setter来操作。
7. 方法可省略return关键字，自动检索最后一行的结果作为返回值。
8. 空值比较不会有NullPointerException异常抛出。

### Groovy高级特性

1. assert断言：可以使用assert代替之前Java的断言语句。
2. 可选类型：可使用类JavaScript的若类型，可以使用def来表示任意类型。
3. 方法调用：调用带参方法时可以省略括号。
4. 字符串定义：字符串定义有三种方式，单引号，双引号，三个单引号。
5. 集合API：集合的定义和使用更加简单，API和Java有所不同，但兼容Java API。
6. 闭包：Groovy的一大特性，跟方法类似的代码块，可赋给一个变量也可以作为参数传递给一个方法，像普通方法一样调用。（类似于Java的Lambda）

## Gradle优点

1. 能实现自动打包。
2. 不再像maven那样的xml语言。
3. 导入插件的语言更简洁

## Gradle进行项目发布

Gradle项目执行前后可以执行相关方法。





## gretty应用

1. 选择servlet 容器
2. 热部署
3. 快速加载，监听`webapp/`中的内容，文件发生改变，无需重启。
4. HTTPS 支持
5.  转发
6. 远程调试（Debug）
7. 将项目生成像tomcat一样，点击就能运行的项目
8. archiveProduct，压缩项目（.zip）









###