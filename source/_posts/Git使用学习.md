---
title: Git使用学习
date: 2018-09-11 15:10:49
tags: 版本管理
---

## 分支相关

 branch 查看当前分支

### 查看本地和远端所有分支

git branch -a

工作区	暂存区	版本库	三种状态

push.  

push origin master 

其中origin是指的远程仓库，master 是本地仓库默认名称

### 新建分支

1. master分支下  pull下最新的代码
2. git checkout -b  <分支名>    :在本地新建分支
3. git push origin <分支名>   :向远端提交分支
4. git branch --set-upstream-to=origin/<分支名>  <分支名>     ：设置head为远程仓库的master
5. git status

### 提交代码到分支

1. git checkout 分支名
2. git	add 文件名				：添加文件到缓存
3.  git rm —cached  :从缓存中去除文件
4. git commit :提交
5. git push :推送到远程分支

### 合并分支与master的代码

1. git branch -vv :本地分支跟踪的远程分支
2. git fetch origin --prune :GitLab更新远程分支信息

### git fetch

```shell
## 在本地新建一个temp分支，并将远程origin仓库的master分支代码下载到本地temp分支；
$ git fetch origin master:temp

## 比较本地代码与刚刚从远程下载下来的代码的区别；
$ git diff temp

## 合并temp分支到本地的master分支;
$ git merge temp

## 如果不想保留temp分支，删除;
$ git branch -d temp
```

#### 直接使用 git fetch 命令

- 创建并更新本地远程分支。即创建并更新origin/xxx 分支，拉取代码到origin/xxx分支上;
- 在FETCH_HEAD中设定当前分支-origin/当前分支对应，如直接到时候git merge就可以将origin/abc合并到abc分支上

#### git fetch origin

- 手动指定了要fetch的remote。在不指定分支时通常默认为master；

#### 对比git pull

与git pull相比git fetch相当于是从远程获取最新版本到本地，但不会自动merge。如果需要有选择的合并git fetch是更好的选择。效果相同时git pull将更为快捷;

### 回滚到指定commit，丢弃commit之后的提交记录

查看历史提交：  git log | head -n 40

1、git reset --hard 139dcfaa558e3276b30b6b2e5cbbb9c00bbdca96   回滚到某个版本

2、git push -f -u origin <分支名>   ：强制提交到分支，覆盖远程分支



### git add 添加了不需要的文件，想撤销上一步操作

`git reset HEAD`

```
git clean -df    放弃本次所有的修改
```

后面什么都不跟的话 就是上一次add里面的全部撤销掉

`git reset HEAD xxxx.java`就是对某个文件进行撤销

### 删除分支以及远程分支

### **删除远程分支**

git push origin --delete  <--分支名-->

git branch -a	//查看分支

### **删除本地分支**

git branch -D <—分支名-->

## 用户信息相关设置

设置提交时显示的用户名称

- 全局设置: git config --global user.name <目标用户名>

​				 git config --global user.email <目标邮箱名>

- 修改当前project 

  git config user.name <目标用户名>

  git config user.email <目标邮箱>

## 上传本地项目到码云

1、在本地项目目录中使用 git init 命令   ---初始化一个git 本地仓库

2、使用git remote add origin  <远程仓库地址>

3、git push -u origin master

## 添加到了.ignore文件但是不生效

无效的原因：对应的目录或者文件已经被git跟踪，此时再加入.ignore后就无效了。

解决方法：

先执行

[文件夹] git rm -r --cached .idea

[文件] git rm --cached demo.iml

再重新加入.ignore文件


