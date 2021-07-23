---
title: "Bash基础教程"
description: 
date: 2021-07-18T20:38:56+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: false
---

## 内容概要

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

## bash介绍和基本格式

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

## 变量

### 变量的名称

变量的名称只能包含0-9，a-z，A-Z以及_，开头不能为数字，一般变量的名称是大写的

### 变量定义

语法格式



​	variable_name=variable_value

例如：

​	NAME="foo"

### 使用变量

通过$符来使用变量，下面的例子定义了NAME变量的值为foo，并通过echo输入NAME的值

```bash
#!/bin/bash
NAME="foo"
echo $NAME
echo ${NAME}
```



### 只读变量

通过`readonly`可以把变量标记为只读的

```bash
#!/bin/bash
NAME="foo"
readonly NAME
# 下面这行会报错
NAME="bar"
```

### 删除变量

通过`unset`命令来删除变量

```bash
#/bin/bash
NAME="foo"
unset NAME
# 什么也不会输出
echo $NAME
```

### 变量的类型

1. 局部变量

   局部变量在脚本中定义，只在当前shell有效

2. 环境变量

   环境变量通过`export variable_name=variable_value`来定义，在所有的子程序中都能访问

3. shell特殊变量

   shell自身设置的一些特殊变量

## 特殊变量

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



## 数组

### 数组定义

```bash
#!/bin/bash
NAME[0]="zero"
NAME[1]="one"
NAME[2]="two"
# bash的初始化语法
NAME=(zero one two)
```

### 使用

通过`${array_name[index]}`来获取数组指定下标的值

```bash
#!/bin/bash
NAME=(zero one two)
echo "第一个下标：${NAME[0]}"
echo "第二个下标：${NAME[1]}"

# 获取数组所有的值
echo "*方法：${NAME[*]}"
echo "@方法：${NAME[@]}"

# 循环
for TOKEN in ${NAME[*]}
do
    echo $TOKEN
done
```

## 操作符

大致分为以下几类操作符

+ 算术
+ 关系
+ 布尔
+ 字符
+ 文件测试

### 算术操作符

需要依靠`expr`命令来实现，操作符合表达式之间必须有空格，`2+2`是错误的，`2 + 2`才是对的

```bash
#!/bin/bash
# + 加
expr 2 + 2
# - 减
expr 2 - 2
# * 乘
expr 2 * 2
# / 除
expr 2 / 2
# % 取余
expr 2 % 2
# = 赋值
a=1
b=$a
# == 判断是否相等，左右必须有空格
[ $a == $b ]
# != 不相等
[ $a != $b ]
```

### 关系操作符

```bash
#!/bin/bash
a=10
b=20
# -eq 两个变量相等为true
[ $a -eq $b ]
# -ne 两个变量不相等为false
[ $a -ne $b ]
# -gt 左边变量大于右边变量则为true
[ $a -gt $b ]
# -lt 左边变量小于右边变量则为true
[ $a -lt $b ]
# -gt 左边变量大于等于右边变量则为true
[ $a -ge $b ]
# -lt 左边变量小于等于右边变量则为true
[ $a -le $b ]
```

### 布尔操作符

```bash
#!/bin/bash
a=10
b=20
# ! 非
[!false]
# -o 或
[ $a -lt 20 -o $b -gt 100 ]
# -a 且
[ $a -lt 20 -a $b -gt 100 ]
```

### 字符操作符

```bash
#!/bin/bash
a="abc"
b="efg"
# = 相等为true
[ $a = $b ]
# != 不相等为true
[ $a != $b ]
# -z 字符串为空为true
[ -z $a ]
# -n 字符串非空为true
[ -n $a ]
```

### 文件测试

```bash
#!/bin/bash
# -e 路径存在为真
[- e 路径]
# -f 路径存在且为普通文件则为真
[ -f 路径]
# -d 路径存在且为目录则为真
[ -d 路径]
# -r 路径存在且可读则为真
# -w 路径存在且可写则为真
# -x 路径存在且可执行则为真
# -s 路径存在且至少有一个字符则为真
# -c 路径存在且为字符型特殊文件则为真
# -b 路径存在且为块特殊文件则为真
```

## 条件语句

+ if...fi

  ```bash
  #/bin/bash
  a="one"
  b="two"
  if [ $a = $b ]; then
  	echo "a等于b"
  fi
  ```

  

+ if...else...fi

  ```bash
  #/bin/bash
  a="one"
  b="two"
  if [ $a = $b ]; then
  	echo "a等于b"
  else
  	echo "a不等于b"
  fi
  ```

  

+ if...elif...else...fi

  ```bash
  #/bin/bash
  a="one"
  b="two"
  c="three"
  if [ $a = $b ]; then
  	echo "a等于b"
  elif [ $a = $c ]; then
  	echo "a等于c"
  else
  	echo "a不等于b且不等于c"
  fi
  ```

+ case...esac

  服务管理

  ```bash
  #!/bin/bash
  case "$2" in
  	"start")
  	systemctl start $1
  	;;
  	"stop")
  	systemctl stop $1
  	;;
  	"status")
  	systemctl status $1
  	;;
  esac
  
  ```

  测试路径是文件还是目录

  ```bash
  #!/bin/bash
  option="${1}" 
  case ${option} in 
     -f) FILE="${2}" 
        echo "File name is $FILE"
        ;; 
     -d) DIR="${2}" 
        echo "Dir name is $DIR"
        ;; 
     *)  
        echo "`basename ${0}`:usage: [-f file] | [-d directory]" 
        exit 1 # Command to come out of the program with status 1
        ;; 
  esac 
  ```

## 循环

+ while

  ```bash
  #!/bin/bash
  a=0
  while [ "$a" -lt 10 ]
  do
  	echo "$a"
  	a=`expr $a + 1`
  done
  ```

  

+ for

  ```bash
  #!/bin/bash
  for TOKEN in $*
  do
     echo $TOKEN
  done
  ```

  

## 转义

+ echo -e

  echo加上-e可以对结果进行转义

  ```bash
  #!/bin/bash
  # 输出\
  echo -e "\\"
  # 输出\\
  echo  "\\"
  ```

  

+ ``

  在``之间的命令会被执行

  ```bash
  #!/bin/bash
  DATE=`date`
  echo "$DATE"
  echo `date`
  ```

  

## 引号

+ 单引号

  单引号内的特殊字符没有含义

  ```bash
  #!/bin/bash
  DATE=`date`
  # 输出DATE变量的值
  echo "$DATE"
  # 输出$DATE
  echo '$DATE'
  ```

  

+ 双引号

  双引号内的特殊字符有含义，并且会执行

  ```bash
  #!/bin/bash
  # 输出date命令的结果
  echo "`date`"
  # 输出`date`
  echo '`date`'
  ```

  

## io重定向

```bash
#!/bin/bash
# > 重定向到文件，覆盖
echo "test" > test.txt
# >> 重定向到文件，追加
echo "test" >> test.txt
# < 读取文件
while read line
do
	echo $line
done  < test.txt

# | 管道
ps -ef | grep sshd
```

## 函数

示例

```bash
#!/bin/bash
# 定义Hello函数，结果为输出Hello World
Hello() {
	echo "Hello World"
}
# 调用Hello函数
Hello
```

### 函数参数

函数的参数获取和脚本类似，都是通过$1 ...来获取

```bash
#!/bin/bash
# 定义加法函数
Add() {
	r="expr $1 + $2"
	echo "$r"
}
# 调用
Add 1 2
```

### 返回结果

使用`return`来返回函数结果，在调用函数后使用$?来获取结果

```bash
#!/bin/bash
# 定义加法函数
Add() {
	r="expr $1 + $2"
	return r
}
# 调用
Add 1 2
# 打印结果
echo $?
```

