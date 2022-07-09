# Makefile 笔记

## 1. 语法规则

**一条规则：**

> 目标：依赖
>
> <tab insert> 命令

Makefile基本规则三要素：

1）目标：

* 通常是要产生的文件名称，目标可以是可执行文件或其它obj文件，也可是一个动作的名称

2）依赖：

* 用来输入从而产生目标的文件
* 一个目标通常拥有多个依赖，也可以**没有**

3） 命令：

* make执行的动作，一个规则可以含几个命令（可以没有）
* 有多个命令时，每个命令各占一行，用tab进行缩进

## 2. 命令格式

make是一个命令工具，它解释Makefile 中的指令（应该说是规则）。

make命令格式：

> make [ -f file ][ options ][ targets ]

1.[ -f file ]：

- make默认在工作目录中寻找名为GNUmakefile、makefile、Makefile的文件作为makefile输入文件
- -f 可以指定以上名字以外的文件作为makefile输入文件

2.[ options ]:

* -v： 显示make工具的版本信息
* -w： 在处理makefile之前和之后显示工作路径
* -C dir：读取makefile之前改变工作路径至dir目录
* -n：只打印要执行的命令但不执行
* -s：执行但不显示执行的命令

3.[ targets ]：

- 若使用make命令时没有指定目标，则make工具默认会实现makefile文件内的第一个目标，然后退出
- 指定了make工具要实现的目标，目标可以是一个或多个（多个目标间用空格隔开）。

## 3. 变量

变量名规则：

* makefile变量名可以以数字开头
* 变量是大小写敏感的
* 变量一般都在makefile的头部定义
* 变量几乎可在makefile的任何地方使用

### 3.1 自定义变量

1）自定义

```
varname=value
```

2）引用变量

````var
$(varname)
````

### 3.2 系统变量

除了使用用户自定义变量，makefile中也提供了一些变量（变量名大写）供用户直接使用，我们可以直接对其进行赋值。

> CC = gcc #arm-linux-gcc
>
> CPPFLAGS : C预处理的选项 如:-I
>
> CFLAGS: C编译器的选项 -Wall -g -c
>
> LDFLAGS : 链接器选项 -L -l

### 3.3 自动变量

- $@: 表示规则中的**目标**
- $^:  表示规则中的**所有条件**, 组成一个列表, 以空格隔开,如果这个列表中有重复的项则消除重复项。
- $<:  表示规则中的**第一个条件**

**tips: 自动变量只能在规则的命令中中使用**

## 4. 模式匹配

示例：.c 生成 目标代码.o文件

```
%.o : %.c
	$(CC) $(CFLAGS) %< -o %@
```

## 5. 函数

makefile中的函数有很多，在这里给大家介绍两个最常用的。

>1. wildcard – 查找指定目录下的指定类型的文件
>
>src = $(wildcard *.c) //找到当前目录下所有后缀为.c的文件,赋值给src
>
>2. patsubst – 匹配替换
>
>obj = $(patsubst %.c,%.o, $(src)) //把src变量里所有后缀为.c的文件替换成.o

```
# 读取当前目录下所有 .c文件
SRC=$(wildcard *.c)
# .c 文件 转换为 .o文件
OBJS=$(patsubst %.c, %.o, $(SRC))

TARGET=test
$(TARGET):$(OBJS)
    gcc $(OBJS) -o $(TARGET) 
%.o:%.c
    gcc -c $< -o $@
```

## 6. Makefile中的伪目标

**伪目标声明:** **.PHONY:clean**

声明目标为伪目标之后，makefile将不会该判断目标是否存在或者该目标是否需要更新

```
.PHONY:clean
clean:
    rm -rf $(OBJS) $(TARGET)
```

## 7. 特殊符号

- “-”此条命令出错，make也会继续执行后续的命令。如:“-rm main.o”
- “@”不显示命令本身,只显示结果。如:“@echo clean done”

