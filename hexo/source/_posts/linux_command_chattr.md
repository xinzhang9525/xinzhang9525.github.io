---
title: Linux chattr 命令详解
excerpt: chattr命令用法详细介绍，实例演示
index_img: /img/post/daibing/movies-first-part.jpeg
date: 2021-08-06 9:00:00
categories: [Linux 常用命令详解]
comments: true
---
### 描述

>chattr命令用于更改Linux文件的属性，比如设置文件为不可删除、只读等类型。只有具有sudo权限的用户才能执行这项命令。chattr可看成是change attributes的缩写：ch+attr
>
>lsattr可查看chattr命令执行后的文件属性



### 语法

```bash
chattr [ -RVf ] [ -v version ] [ mode ] files
```



### 参数

```bash
可选参数：[ -RVf ] 和 [-v version]
-R: 递归处理，将指定目录下的所有文件及子目录一并处理.
-V: 显示输出指令执行过程
-f: 若该文件权限无法被更改也不要显示错误讯息
-v version: 设置文件或目录版本


必选参数：[ mode ]
+<属性>：开启文件的该项属性
-<属性>：关闭文件的该项属性
=<属性>：使得文件只有该项属性


属性：
a: 针对文件：只允许程序在文件后面追加数据。针对文件夹：只允许创建和修改文件内容，但不允许删除
A: 告诉系统不修改该文件的最后访问时间，以减小磁盘I/O负载
b: 不更新文件的最后存取时间
c: 将文件或目录压缩后存放
C: 不执行写入时复制，多个用户对同一资源进行写操作时，不生成副本
d: 当dump程序执行时，该文件或目录不会被dump备份
D: 同步更新文件或目录
e: 使用extent文件格式
i: 不得任意改动文件或目录
j: 数据在写入到文件前暂存到journal
s: 保密性删除文件或目录
S: 及时更新文件或目录
t: 不支持文件尾部合并
T: 使得目录下的文件存放位置不存在联系，因为一般而言，同一目录下的文件在硬盘上的存放位置越相邻越好
u: 预防意外删除

```



### 示例大全

#### 设置文件为只读、不可删除

```bash
# test.txt文件
sudo chattr +i test.txt
```

#### 设置目录下的所有文件为只读、不可删除

```bash
# home目录
sudo chattr -R +i /home/
```

#### 设置文件为只能追加、不可删除

```bash
sudo chattr +a test.txt
```

#### 设置目录下的所有文件只能追加、不可删除

```bash
sudo chattr -R +a /home/
```



#### 取消文件的只读、不可删除

```bash
# test.txt文件
sudo chattr -i test.txt
```

#### 取消目录下的所有文件只读、不可删除

```bash
# home目录
sudo chattr -R -i /home/
```

#### 取消文件的只能追加、不可删除

```bash
sudo chattr -a test.txt
```

#### 取消目录下的所有文件只能追加、不可删除

```bash
sudo chattr -R -a /home/
```



#### 查看文件属性

> 使用lsattr命令

```bash
# 查看test.txt文件的属性
lsattr test.txt

# 查看当前目录下所有文件的属性
lsattr ./

# 查看所有以wav后缀结尾的文件的属性
lsattr *.wav
```



### 注意

> chattr 并不适用于所有目录，chattr不能保护/, /dev, /var目录
