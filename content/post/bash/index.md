---
title: "Bash教程"
description: 
date: 2021-07-18T20:38:56+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: false
---

# 内容概要

+ bash介绍和基本格式
+ 变量
+ 特殊变量
+ 数组
+ 操作符
+ 条件语句
+ 循环语句
+ 转义
+ 引号
+ io重定向
+ 函数

# bash介绍和基本格式

shell给用户和linux系统之间提供了接口，它搜集用户的输入并执行，执行完成后输出结果。shell是一个环境，用户可以执行命令，程序和shell 脚本。不同的操作系统有不同的shell发行，语法也不尽相同。

bash被广泛的使用，是大多数操作系统默认的shell，下面通过一个例子来介绍一下bash的基本格式



1. 新建一个文件，以.sh结果

   ```bash
   touch example.sh
   ```

   

2. 使用vim编辑文件

   ```bash
   vim example.sh
   ```

3.  在文件中输入以下内容

   ```bash
   #!/bin/bash
   # Author: xingxiaowen
   echo "What is your name?"
   read PERSON
   echo "Hello, $PERSON"
   ```

4. 退出vim 编辑并给example.sh添加执行权限

   ```bash
   # 退出vim
   esc
   :wq
   # 添加可执行权限
   $ chmod +x ./example.sh
   ```

5. 执行example.sh脚本

   ```bash
   $ ./example.sh
   What is your name?
   Foo
   Hello, Foo
   ```

文件的第一行`#!/bin/bash`是告诉系统使用bash来执行该脚本; 第二行`# Author: xingxiaowen` 是注释，shell脚本中注释以#开头;第三行`echo "What is your name?"是执行echo程序（echo是一个输出字符的命令）`，意思是直接输出`What is your name`;第四行`read PERSON`执行read命令，该命令等待用户控制台输入并把用户输入的内容赋给PERSON变量;最后一行`echo "Hello, $PERSON"`同样是执行字符输出，只是引用了上一步的PERSON变量。

# 变量

## 变量的名称

变量的名称只能包含0-9，a-z，A-Z以及_，开头不能为数字，一般变量的名称是大写的

## 变量定义

语法格式



​	variable_name=variable_value

例如：

​	NAME="foo"

## 使用变量

通过$符来使用变量，下面的例子定义了NAME变量的值为foo，并通过echo输入NAME的值

```bash
#!/bin/bash
NAME="foo"
echo $NAME
```



## 只读变量

通过`readonly`可以把变量标记为只读的

```bash
#!/bin/bash
NAME="foo"
readonly NAME
# 下面这行会报错
NAME="bar"
```

## 删除变量

通过`unset`命令来删除变量

```bash
#/bin/bash
NAME="foo"
unset NAME
# 什么也不会输出
echo $NAME
```

## 变量的类型

1. 局部变量

   局部变量在脚本中定义，只在当前shell有效

2. 环境变量

   环境变量通过`export variable_name=variable_value`来定义，在所有的子程序中都能访问

3. shell特殊变量

   shell自身设置的一些特殊变量

# 特殊变量

```bash
#/bin/bash
# 当前shell文件名
echo $0  
# 脚本输入参数$1,$2...
echo $n  
# 输入参数个数
echo $#  
# 所有参数加上双引号　"$1 $2 $3"
echo $*
for TOKEN in $*
do
   echo $TOKEN
done
# 给所有参数单独加双引号 "$1" "$2"
echo $@  
# 最后一个命令状态 0 successful 1 unsuccessful
echo $?  
# 当前脚本进程id
echo $$  
# 最后一个后台命进程id
echo $!  
```

