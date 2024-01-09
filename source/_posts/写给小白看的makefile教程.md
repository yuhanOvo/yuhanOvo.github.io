---
title: 写给小白看的makefile教程（入门）
date: 2021-08-10 16:44:00
updated: 2021-08-10 16:44:00
categories: 
  - linux学习
tags:
  - linux
  - c++
  - c
  - makefile
---

makefile类似于shell脚本，它能让terminal执行一系列的命令，在Linux下写c++时，我们可以用到makefile来帮助我们进行预处理、编译、汇编和链接。

下面就来说说如果使用makefile吧~

<!--more-->

## 规则

话不多说，直接上makefile的**一条规则**的语法格式：

```
目标：依赖
【Tab键缩进】命令
```

一条**规则**一共分为**目标**、**依赖**、**Tab缩进**、**命令**四个部分。

**目标：** 是我们希望生产的目标文件

**依赖**： 是我们生产目标文件的源文件（依赖文件）

**Tab缩进：** 语法规则，必须要有

**命令：** 从依赖文件生成目标文件的命令

---

举例子来说明上面的东西：

在当前所工作的目录下创建文件名为makefile的文件即可，比如：

![image-20210810155500869](https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/makefile入门教程/image-20210810155500869.2ymjbwx8roc0.png)

在makefile里面编辑如下：
![image-20210810155741023](https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/makefile入门教程/image-20210810155741023.59ie1n1juqs0.png)

在terminal中敲入make即可：

![image-20210810155957835](https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/makefile入门教程/image-20210810155957835.3u1sfqagx080.png)

可以看到在敲入make按下回车后，terminal自动帮我执行了 g++ hello.cpp -o hello 这行命令。

回到makefile文件内容：
![image-20210810160235006](https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/makefile入门教程/image-20210810160235006.3v4olf8my8g0.png)

在执行make命令时，系统检查到makefile文件中的第一行内容的目标文件为 hello ，为了生成 hello 这个文件它需要一个依赖文件叫 hello.cpp , 此时它检查到当前目录中存在 hello.cpp ， 那么接下来就只需要执行第二行的命令产生目标文件就可以了。

## 原理

makefile会将它首次遇到的目标文件视为**最终目标文件**，如若生成目标文件的依赖文件不存在，则检查是否有其它规则生成此依赖文件。

---

这句话什么意思呢，让我们再举例子说明一下：
我们重新编辑makefile文件为：

![image-20210810162235389](https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/makefile入门教程/image-20210810162235389.6wriil40uug0.png)

makefile在第一行读到目标文件 hello ， 这是它第一个读到的目标文件，它将这个目标文件视为它需要生成的最终目标文件，生成hello目标文件需要hello.o的依赖文件。

可是当前目录并不存在该文件，于是makefile检查是否还有其它规则来生成该依赖文件，读到第三行的时候，makefile发现了有一个规则能生成hello.o文件。

而生成该hello.o文件的依赖文件为hello.cpp，makefile检查当前目录确实存在hello.cpp文件，于是先执行`g++ -c hello.cpp -o hello.o`生成hello.o，再执行`g++ hello.o -o hello` 生成名为hello的最终目标文件。

---

此时再读这句话我相信你能理解这句话的含义了。

那么，如果我将3、4行和1、2行的内容对调，即：

![image-20210810163145200](https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/makefile入门教程/image-20210810163145200.20oljvyrwleo.png)

会怎么样呢？

按照makefile的原理，它只会生成到hello.o就结束：

![image-20210810163409428](https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/makefile入门教程/image-20210810163409428.3rmndgf17mw0.png)

makefile肯定有相应的语法来指定这个我们想生成的最终目标文件啦：

![image-20210810163544235](https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/makefile入门教程/image-20210810163544235.5ukrcukyhcc0.png)

但是，此时指定在最末行这行语句会生效吗？

答案是**不会**。

makefile仍然还是会读取最开始遇到的目标文件视为最终目标文件。

因此，我们将这行指定最终文件的语句放到所有规则的上面，就怎么也不会出问题了：

![image-20210810164013746](https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/makefile入门教程/image-20210810164013746.3umv04pvueg0.png)

## 函数

### wildcard函数

如下：

```makefile
src = $(wildcard *.cpp)
```

该函数的作用是获取当前目录下的所有后缀名为.cpp的文件并将其赋值给src，也就是变量src是当前目录下后缀为.cpp文件的集合。

---

举个例子：

在当前目录下

![image-20210815125345013](https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/makefile入门教程/image-20210815125345013.1omkhviru8o0.png)

在terminal中输入make，makefile中的 `src = $(wildcard *.cpp)` 会获取到当前目录中后缀为.cpp的文件，在这个例子中，

src = [add.cpp ,  main.cpp,  sub.cpp]

#### 文件目录

当然，wildcard函数可以获取任何路径下的某一种后缀名的文件列表。

比如：

```makefile
src = $(wildcard ./src/*.c)
```

获取当前目录下的src目录下的所有.c后缀的文件列表。



后缀可以是任何你想要的后缀，目录也可以是任何你想要的目录。下面的所有函数和变量同样可以应用。

### patsubst函数

如下：

```makefile
obj = $(patsubst %.cpp, %.o, $(src))
```

如果需要引用变量，只需要在 **$()** 中加入该变量就可以了。

该函数一共有三个参数，第一个参数是需要转换格式的文件列表，第二个参数是想转换成的格式的文件列表，第三个参数是第一个参数的依赖。

---

举例子说明：

在上一个例子中，src = [add.cpp ,  main.cpp,  sub.cpp]，则patsubst函数会将src中所有符合后缀名为.cpp的文件列表获取，用 **%.cpp** 来表示，而将其改为后缀名为 .o 的文件，由 .o 文件构成的文件列表用即为变量 **obj**。

也就是说：

src  =  [add.cpp,  main.cpp,  sub.cpp]

obj = [add.o, main.o, sub.o]

## 编译规则

至此，我们简单编译多个c++文件所需要的全部文件名都已经准备好了，接下来就是规则怎么写的问题了。

先看如下语句：

```makefile
src = $(wildcard *.cpp)
obj = $(patsubst %.cpp, %.o, $(src))
ALL:main

main:$(obj)
	g++ $^ -o $@

$(obj):%.o:%.cpp
	g++ -c $< -o $@ 
```

前三行不再解释。

先来解释两个变量， **$^** 表示该规则的依赖， **$@** 表示该规则的目标。

在第五行的规则中， **$^**  表示 **$(obj)** ,  **$@** 表示 **main**。

第六行 `g++ $^ -o $@` 等于 `g++  add.o, main.o, sub.o -o main` 。



第八行的含义是获取 **$(obj)** 中的后缀为.o的文件，将其作为该规则的目标文件（列表）。

而该目标文件（列表）的依赖则为将 **$(obj)** 中的后缀为.o的文件的后缀.o替换成.cpp的文件（列表）。

**$<** 同样代表依赖，但和 **$^** 不同， **$<** 会遍历依赖中的文件，单个输出。有点像python里面的遍历列表的意思。

也就是说第九行的 `g++ -c $< -o $@ `等于

```
g++ -c add.cpp -o add.o
g++ -c main.cpp -o main.o
g++ -c sub.cpp -o sub.o 
```

## clean函数

当你再次在terminal中输入make命令时，makefile会检查你是否对源文件进行了修改来决定是否需要重新编译部分步骤，如果你没有修改源文件，makefile将不会进行编译，并告诉你：无需做任何事。

如果你需要删除某些文件可以使用makefile的clean函数

makefile中clean函数格式如下：

```makefile
clean:
	-rm -rf $(obj) main # 可以加入任何你想删除的文件
```

配置好后，只需在terminal中输入`make clean`即可。

## 结尾

至此，一个初级的makefile文件如下：

```makefile
src = $(wildcard *.cpp)
obj = $(patsubst %.cpp, %.o, $(src))

ALL:main

main:$(obj)
	g++ $^ -o $@

$(obj):%.o:%.cpp
	g++ -c $< -o $@

clean:
	-rm -rf $(obj) main
```

希望对你有所帮助。
