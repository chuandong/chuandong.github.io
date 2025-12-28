title: win
date: 2024-04-28
tags: win
description: win study

### 1.bat文件内容命令

```
@echo off :表示关闭命回显
color 0a :表示设置cmd展示的字体颜色,可以使用 color ? 查看颜色种类
title bat file write :表示bat文件的标题
echo =========== :表示输出的内容
pause :表示暂停
del :表示删除
cls :表示清屏
e: :表示进入E盘
rd . /s/q :表示进入到哪个盘格式化哪个盘
:d :表示d这个区间
start :表示打开cmd命令窗口
goto :表示跳转到哪个区间去 
set /p num=请输入你的选择 :表示读取输入的内容
shutdown -s -f -t %num% :表示读取num的值，并关机
if "%num%"=="1" goto 1 :表示跳转到1区间
md：表示创建文件夹
netstat -na：查看本机开放的所有端口
```

```bat_study.bat
@echo off
color 0a
titie bat file write
echo ==========================
echo     welcome my world
echo ==========================
pause
```

-- 这是表示bat文件的内容

### 2. cmd命令

```
net user 11655： 查看11655的用户信息
net user abc /del: 删除abc用户
net localgroup: 本地组
mstsc：登入远程命令
services.msc:打开服务窗口
```

### 3. 用户管理命令

```
net user                #查看用户列表
net user 用户名 密码      #改密码
net user 用户名 密码 /add #创建一个新用户
net user 用户名 /del     #删除一个用户
net user 用户名 /active:yes/no #激活或禁用账户
```

### 4.内置组

```
1）administrators  #管理员组
2）guests          #来宾组
3）users           #普通用户组
4）network         #网络配置组
5）print           #打印机组
6）remote desktop  #远程桌面组
```

