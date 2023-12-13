# Git

[TOC]

## Git配置

```shell
#查看配置
git config -l
#查看用户配置
git config --global  --list
#查看系统配置
git config --system --list
#设置用户名与邮箱（用户标识，必要）
git config --global user.name "名称"
git config --global user.email "邮箱"
```

## 基本理论

![image-20230429105208102](/Users/chuxin/Desktop/Linux学习笔记/图片/image-20230429105208102.png)

## 项目搭建

### 本地仓库搭建

```shell
# 在当前目录新建一个Git代码库
$ git init
```

### 克隆远程仓库

```shell
# 克隆一个项目和它的整个代码历史(版本信息)
$ git clone [url]  # https://github.com/xxx
```

## 忽略某些文件

新建.gitignore文件，在该文件中添加要忽略的文件名(*.out)

## 基本命令

```shell
git status #查看状态
git add . #暂存所有文件(工作区->暂存区)
git commit -m "注释" #提交暂存区的文件(暂存区->本地仓库)
git log [option] #查看日志
	--all #显示所有分支
	--pretty=oneline #显示为一行
	--abbrev-commit #使commitid简短
	--graph #以图的形式显示
git reflog #查看精简日志
git reset --hard commitid #版本回退
```

## 分支

```shell
git branch #查看分支
git branch 分支名 #创建分支
git checkout 分支名 #切换分支
git checkout -b 分支名 #创建并切换到该分支
git merge 分支名 #将目标分支合并到当前分支
git branch -d 分支名 #删除分支（-D强制删除）
```

## 远程仓库

```shell
git remote -v #查看别名
git remote add 别名 地址 #创造别名
git push 别名 分支名 #推送到远程仓库 -f强制（不同步）
git pull 别名 分支名 #拉取远程分支到本地
git clone 地址 #克隆远程仓库
```

## 跨团队协作

> 以下功能在GitHub上操作，非命令行

### Fork 

在GitHub上将一个项目复制一份到自己的账号

### Pull Request

在GitHub上用户向项目贡献者发起的一种请求，请求对方接受自己所做的修改并将其合并到原项目

## SSH密钥（免密登陆）

1. 查看是否已经有了ssh密钥：cd ~/.ssh
   如果没有密钥则不会有此文件夹，有则备份删除
2. 生存密钥：

> $ ssh-keygen -t rsa -C “邮箱”
> 按3个回车，密码为空。

3.  在～/.ssh目录下cat id_rsa.pub
4. 将内容(ssh key)添加到GitHub账号或仓库