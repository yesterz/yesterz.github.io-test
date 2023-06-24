---
title: "玩转 Git 三剑客"
excerpt: "Page not found. Your pixels are in another canvas."
sitemap: false
permalink: /GGit
---
玩转 Git 三剑客

记录 GitHub Pages 搭建学习 git

Pages 的主题文档地址：https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/

# Ch1 Git 基础

## 01-课程综述

Git 是版本控制系统（VCS）

![image-20230624105057547](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624105057547.png)

### 集中式 VCS：

1. 有集中的版本管理服务器
2. 具备文件管理和分支管理能力
3. 集成效率有明显地提高
4. 客户端必须时刻和服务器相连

![image-20230624105138425](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624105138425.png)

### 分布式 VCS

1. 服务端和客户端都有完整的版本库
2. 脱离服务端，客户端照样可以管理版本
3. 查看历史和版本比较等多数操作，都不需要访问服务器，比集中式 VCS 更能提高版本管理效率

bitkeep

![image-20230624105447766](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624105447766.png)

### Git 的特点

1. 最优的存储能力
2. 非凡的性能
3. 开源的
4. 很容易做备份
5. 支持离线操作
6. 很容易定制工作流程

![image-20230624105557255](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624105557255.png)

然后商业的平台就出来了。

![image-20230624105621303](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624105621303.png)

GitHub 是全球最大的开源社区。

DevOps 打破开发和运维的壁垒。

## 02-安装 Git

git 的官网 https://git-scm.com/

Window Installation https://git-scm.com/download/win

Linux/Unix Installation https://git-scm.com/download/linux

## 03-使用 Git 之前需要做的最小配置

### 配置 user 信息

![image-20230624110432248](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624110432248.png)

配置 user.name 和 user.email

```
git config --global user.name 'your_name'
git config --global user.email 'your_email'@domain.com
```

### config 的三个作用域

![image-20230624110733986](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624110733986.png)

缺省等同于 local

```shell
git config --local    # local 只对某个仓库有效
git config --global   # global 对当前用户所有仓库有效
git config --system   # system 对系统所有登录的用户有效
```

显示 config 的配置，加 --list

``` shell
git config --list --local
git config --list --global
git config --list --system
```

![image-20230624111229509](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624111229509.png)

## 04-创建第一个仓库并配置 local 用户信息

### 建 Git 仓库

![image-20230624112354196](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624112354196.png)

两种场景：

1. 把已有的项目代码纳入 Git 管理
2. 新建的项目直接用 Git 管理

![image-20230624113143896](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624113143896.png)

```she
pwd
ls -al
git init git_learning
cd git_learning
ls -al
git config --global --list
git config --local user.name 'suling'
git config --local user.email 'so_ling@163.com'
git config --local --list
```

![image-20230624113109811](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624113109811.png)

``` shell
pwd
cp ../0-material/readme .
git commit -m'Add readme'
git add readme
git status
git commit -m'Add readme'
git log
```

### 本讲内容

1. 用 git init 创建了第一个仓库
2. global 和 local 优先级不同，local 高
3. 直接 commit 是不起作用的，需要先 git add 文件。

## 05-通过几次 commit 来认识工作区和暂存区

### 往仓库里添加文件

![image-20230624112002526](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624112002526.png)

![image-20230624114232297](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624114232297.png)

``` shell
pwd
ls -al
cp ../0-material/index.html.01 index.html
cp -r ../0-material/images .
git status
git add index.html images
ls -al
git status
git commit -m'Add index + log'
git log
```

![image-20230624114603332](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624114603332.png)

``` shell
git add styles
git status
git commit -m'Add style.css'
git log
...
git add js
git commit -m'Add js'
```

![image-20230624115100966](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624115100966.png)

``` shell
git add -u
git status
git commit -m'Add refering projects'
```

-u 所有工作区当中被 Git 管理的文件一起提交到暂存区

## 06-给文件重命名的简便方法

![image-20230624115624337](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624115624337.png)

``` shell
mv readme readme.md
git status
git add readme.md
git rm readme
git reset --hard
```

![image-20230624115824780](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624115824780.png)

``` shell
git reset --hard
git status
```

![image-20230624115908825](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624115908825.png)

``` shell
git mv readme readme.md
git status
git commit -m'Move readme to readme.md'
```

## 07-通过 git log 查看版本演变历史

![image-20230624120455339](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624120455339.png)

``` shell
git log --oneline
git log -n4
git log -n4 --oneline
git log -n2 --oneline
```

![image-20230624120656599](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624120656599.png)

``` shell
git branch -v
git checkout -b temp 415c5c8086e16399
ls -al
git commit -m'Add test'
git commit -am'Add test'
git branch -av
```

![image-20230624121145966](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624121145966.png)

```shell
git log --all --graph
git log --oneline --all -n4 --graph
git log --oneline --all temp
git log --oneline temp
```

如果想看详细的 git log 指令的文档 `git help --web log`

![image-20230624121428449](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624121428449.png)

## 08-gitk: 通过图形界面工具来查看版本历史

![image-20230624132915492](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624132915492.png)

在仓库的目录下输入 `gitk`

![image-20230624134949887](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624134949887.png)

## 09-探秘 .git 目录









# ch2 独自使用 Git 时的场景











# ch3 Git 与 GitHub 简单同步

全球最大的代码托管平台 [Github](https://www.github.com)

## 30-注册一个 Github 账号

## 31-配置公私钥

## 32-在 GitHub 上创建个人仓库

## 33-把本地仓库同步到 GitHub

![image-20230624143249909](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624143249909.png)

``` shell
git remote -v
git remote add github git@github.com:git201901/git_learning.git
git remote -v
```

![image-20230624145343978](C:/Users/hcho/AppData/Roaming/Typora/typora-user-images/image-20230624145343978.png)

``` shell
git fetch github master
gitk --all
git branch -v
git branch -va
git checkout master
git merge github/master
git merge --allow-untelated-histories github/master
```







# ch6 初时 Github

