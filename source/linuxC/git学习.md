---
title: git学习
date: 2021-10-31 23:03:32
description: git学习内容
tags: git
---

-   设置用户名信息

    git config --global user.name "chuandong"

    git config --global user.email "1165532311@qq.com"

-   查看配置信息

    git config --global user.name

    git config --global user.email

-   创建本地仓库

    ```
    在电脑上创建一个文件夹作为本地仓库；
    进入这个目录，点击右键打开 git bash窗口；
    执行 git init命令；
    如果创建成功可以在目录下看见 .git目录。
    ```

    ```
    git status (查看文件的状态)
    git add . (添加所有的文件)
    git commit -m "提交说明"
    git log [option] （查看提交日志）
    	* --all 显示所有的分支
    	* --pretty=oneline 将提交信息显示为一行
    	* --abbrev-commit 使得输出的commitld更简短
    	* --graph 以图的形式显示
    git reset --hard commitID （版本回退）
    	commitID可以使用 git-log 或git log 查看
    git reflog （查看已经删除的的详细记录）
    创建 .gitignore文件，将不需要提交的文件放在里面，git就不会识别提交
    git branch     （查看本地分支）
    git branch 分支名（创建本地分支）
    git branch -d 分支名 （删除分支）
    git branch -D 分支名（强制删除分支,如果没有合并的分支删除，git会警告，就用此命令）
    git branch -vv (查看对应关系)
    git checkout    （切换分支）
    git checkout -b 分支名 （创建并创建分支）
    git merge （合并分支）切换到当前需要合并过来的分支，例如其它分支合并到master分支上，在master分支上执行此命令。
    ```

    

-   配置ssh公钥

    ```
    生成ssh公钥
    	ssh-keygen -t rsa (不断回车，如果公钥已经存在，则自动覆盖)
    获取公钥
    	cat ~/.ssh/id_rsa.pub
    在github账号里面，点击登入退出里的setting，里面有SSH keys的配置选项，在里面添加就行；
    验证配置是否成功
    	ssh -T git@github.com
    ```

    

-   添加远程仓库

    ```
    创建远程仓管库：
    	git remote add origin git@github.com:chuandong/chuandong.github.io.git
    查看创建的远程仓库：
    	git remote
    推送到远程仓库
    	命令：git push [-f] [--set-upstream][本地分支名：远端分支名]
    	git push  origin master 
    	git push -f origin master(强制覆盖推送，不推荐使用)
    	--set-upstream 推送到远端的同时并且建立起和远端分支的关联关系。
    		git push --set-upstream origin master
    ```

-   克隆仓库

    ```
    git clone git@github.com:chuandong/chuandong.github.io.git Test (Test为指定的文件夹，也可以不加，采用默认的)
    ```

-   从远程仓库中抓取和拉取

    ```
    抓取命令：git fetch [remote name][branch name]
    	*抓取指令就是将仓库里的更新都抓取到本地，不会进行合并；
    	*如果不指定远端名称和分支名，则抓取所有分支。
    拉取命令：git pull [remote name][branch name]
    	*拉取指令就是将远端仓库的修改拉到本地并自动进行合并，等同于fetch+merge；
    	*如果不指定远端名称和分支名，则抓取所有并更新当前分支。
    ```

    

