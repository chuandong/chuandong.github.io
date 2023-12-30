---
title: linux_shell
date: 2022-01-19 17:07:05
description: shell
tags: shell
---

### 1. 加法计算

```shell
num=1
num=$num + 1
echo $num  #输出结果还是 1+1
#第一种写法
num=`expr $num + 1` #expr只用于整数运算，小数不支持, + 需要空格
#第二种写法
let "num += 1" 
let "num= $num + 1" #let只用于整数运算，小数不支持
#第三种写法
((num += 1))  #一样变量
num1 = $((num = $num + 1)) #不一样变量，只用于整数运算，小数不支持
#第四种写法
num=$[$num + 1] #只用于整数运算，小数不支持
```

### 2. 判断文件的类型

```shell
if [ -e  "$file" ]  #如果file存在，返回为真
if [ -d  "$file" ]  #如果file目录存在，返回为真
if [ -f  "$file" ]  #如果file存在并且是普通文件，返回为真
if [ -L  "$file" ]  #如果file是链接文件，返回为真
if [ -r  "$file" ]  #如果file是可读文件，返回为真
if [ -w  "$file" ]  #如果file是可写文件，返回为真
if [ -x  "$file" ]  #如果file是可执行文件，返回为真
if [ -z  "$file" ]  #如果file值为空，返回为真
if [ -n  "$file" ]  #如果file值不为空，返回为真
```

### 3. 运算表示

```shell
-eq #相等
-ge #大于等于
-gt #大于
-le #小于等于
-lt #小于
-ne #不等于
```

### 4. 使用read读取文本行

​	read命令接收标准输入，或其他文件描述符的输入。得到输入后，read命令将数据但放入一个标准变量中。

​	read读取文件时，每次调用read命令都会读取文件中的“一行”文本。当文件没有可读的行时，read命令将以非零状态退出。

```shell
#第一种写法
while read myline
do
	echo "LINE:"$myline
done < myfile.txt
#第二种写法
cat myfile.txt | while read myline
do
	echo "LINE:"$myline
done
```

### 5. 文件描述符

```shell
kill -9 `ps -elf | grep -v grep | grep $1 | awk '{print $4}'` 1>/dev/null 2>&1
# 0 代表标准输入 1 代表标准输出 2 代表错误
# “>” 表示重定向
# 1>/dev/null 表示标准输出重定向到空设备，就是不显示任何信息
# >& 表示等同于的意思 2>&1 表示2的输出重定向等同于1.标准错误输出重定向等同于标准输出，因为之前标准输出已经重定向到了空设备文件，所以标准错误也重定向到空设备中。
# $? 指的是前一个命令的返回码
```

### 6. shell编程debug

```shell
sh -x #显示脚本执行过程，能解决80-95%的问题
```

