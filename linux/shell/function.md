---
title: Bash 函数详解
date: 2016-06-02 13:00:00
updated:
comments: true
tags:
- Linux
- Shell
categories:
- Linux
- Shell
---

默认情况下，脚本中定义的任何变量均为 `全局变量`，可以在函数内访问。

<!--more-->

# 创建函数

```bash

# function关键字创建函数

function func1 {
   echo "this is func1"
}

# 接近其它语言形式的函数

func2() {
   echo "this is func2"
}

# sh 函数命名使用下划线分隔

fun_read(){
  echo "sh function"
}
```

>注意，为了兼容 sh，尽量使第二种方法。

# 引用函数

```bash
func1
func2
```

# 返回值

`return` 只能用来返回整数值（0-255 之间）。

# 变量作用范围

默认情况下，脚本中定义的任何变量均为 `全局变量`，可以在函数内访问。可以使用 `local` 关键字来定义局部变量。

# 位置参数变量

* `"$*"`

被双引号扩住，所有参数被认为是一个字段

for in 循环只会循环一次

* `"$@"`

被双引号扩住，参数会以空格划分开

* `$#`

传入的参数个数。可以为 0

* `$$`

脚本的当前进程 ID 号。

* `shift n`

`n` 默认为 1 ，该命令用于偏移位置参数变量，原来的 $2 变为 $1，以此类推。

# `m=${1:-start}`

如果 $1 存在且不为空，m 就是 $1

如果 $1 不存在或为空，那么 m 就是 start



# 参考链接

* http://www.cnblogs.com/dyllove98/p/3189998.html
