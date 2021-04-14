---
title: Git基础学习
date: 2021-04-14 23:07:30
tags: 
  - Git
categories: 
  - 工具
description: Git
comments: true
top_img: https://cdn.jsdelivr.net/gh/liuyeweiQX/picBed/img/post/img17.jpg
cover: https://cdn.jsdelivr.net/gh/liuyeweiQX/picBed/img/post/img17.jpg
---

### 版本控制
**版本控制**（Revision control）是一种在开发的过程中用于管理我们对文件、目录或工程等内容的修改历史，方便查看更改历史记录，备份以便恢复以前版本的软件工程技术。

1. 实现跨区域多人协同开发
2. 追踪和记载一个或者多个文件的历史记录
3. 组织和保护你的源代码和文档
4. 统计工作量
5. 并行开发、提高开发效率
6. 跟踪记录整个软件的开发过程
7. 减轻开发人员的负担，节约时间，同时降低人为错误

主流版本控制工具：Git、SVN、CVS、VSS、TFS、Visual Studio Online

### Git和SVN的区别
1. 本地版本控制
记录文件每次的更新，可以对每个版本做一个快照，或是记录补丁文件，适合个人使用
{% asset_img content1.png content1 %}

2. 集中版本控制（代表工具：SVN）
所有的版本数据都存在服务器上，用户本地只有自己以前所同步的版本，如果不连网的话，用户就看不到历史版本，也无法切换版本验证问题，或在不同分支工作。而且，所有数据都保存在单一的服务器上，有很大的风险这个服务器会损坏，这样就会丢失所有的数据，当然可以定期备份。
{% asset_img content2.png content2 %}

3. 分布式版本控制
每个人都拥有全部的代码，有安全隐患。
所有版本信息仓库全部同步到本地的每个用户，这样就可以在本地查看所有版本历史，可以离线在本地提交，只需在连网时push到相应的服务器或其他用户那里。由于每个用户那里保存的都是所有的版本数据，只要有一个用户的设备没有问题就可以恢复所有的数据，但这增加了本地存储空间的占用。
不会因为服务器损坏或者网络问题，造成不能工作的情况！
{% asset_img content3.png content3 %}

**Git与SVN最主要区别**
SVN是集中式版本控制系统、版本库是集中放在中央服务器的，而工作的时候，用的都是自己的电脑，所以首先要从中央服务器得到最新的版本，然后工作，完成工作后，需要把自己做完的活推送到中央服务器。集中式版本控制系统是必须联网才能工作，对网络带宽要求较高。
Git是分布式版本控制系统，没有中央服务器，每个人的电脑就是一个完整的版本库，工作的时候不需要联网了，因为版本都在自己电脑上。协同的方法是这样的：比如说自己在电脑上改了文件A，其他人也在电脑上改了文件A，这时，你们两之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。Git可以直接看到更新了哪些代码和文件。
Git是目前世界上最先进的分布式版本控制系统。

### Git环境配置
官网下载：https://git-scm.com/，  下载git对应操作系统的版本。
官网下载太慢，可以使用淘宝镜像下载：http://npm.taobao.org/mirrors/git-for-windows/
安装则一直点击下一步
卸载则先清理环境变量配置，再卸载

- Git Bash：Unix与Linux的风格的命令行，使用最多，推荐最多
- Git CMD：Windows风格的命令行
- Git GUI：图形界面的Git，不建议初学者使用，尽量先熟悉常用命令

基本Linux命令学习：
```
    cd：改变目录
    cd .. ：回退到上一个目录，直接cd进入默认目录
    pwd：显示当前所在的目录路径
    ls(ll)：都是列出当前目录中的所有文件，只不过ll列出的内容更为详细
    touch：新建一个文件 如touch index.js 就会在当前目录下新建一个index.js文件
    rm：删除一个文件，rm index.js 就会把index.js文件删除
    mkdir：新建一个文件夹
    rm -r：删除一个文件夹，rm -r src删除src文件夹
    mv：移动文件，mv index.html src 将index.html文件移动到src目录下
    reset：重新初始化终端/清屏
    clear：清屏
    history：查看命令历史
    help：帮助
    exit：退出
    #表示注释
```

查看配置：git config -l
查看全局配置：git config --global --list（这个事先必须自己先配置好）
{% asset_img content4.png content4 %}
git config --global user.name "liuyeweiQX" #名称
git config --global user.email 76949912@qq.com #邮箱

### Git基本理论
Git本地有三个工作区域：工作目录（Working Directory）、暂存区（Stage/Index）、资源库（Repository或Git Directory）。如果在加上远程的git仓库（Remote Directory）就可以分为四个工作区域。文件在这四个区域之间的转换关系如下：
{% asset_img content5.png content5 %}

- WorkSpace：工作区，就是平时存放项目代码地方
- Index/Stage：暂存区，用于临时存放你的改动，事实上他只是一个文件，保存即将提交到文件列表信息
- Repository：仓库区（或本地仓库），就是安全存放数据的位置，这里面有提交到所有版本的数据。其中HEAD指向最新放入仓库的版本
- Remote：远程仓库，托管代码的服务器，可以简单的认为是项目组中的一台电脑用于远程数据交换

### Git项目搭建
git init：本地项目创建，创建后可以看到项目目录多出了一个.git目录，关于版本等的所有信息都在这个目录里面
git clone + 远程目录：克隆远程仓库，

### Git文件操作
文件4种状态：
- Untracked：未跟踪，此文件在文件夹中，但并没有加入到git库，不参与版本控制，通过git add状态变为Staged
- Unmodify：文件已经入库，未修改，即版本库中的文件快照内容与文件夹中完全一致，这种类型的文件有两种去处，如果它被修改变为Modified，如果使用git rm移出版本库，则成为Untracked文件
- Modified：文件已修改，仅仅是修改，并没有进行其他的操作，这个文件也有两个去处，通过git add可进入暂存staged状态，使用git checked则丢弃修改过，返回unmodify状态，这个git checkout即从库中取出文件，覆盖当前修改
- Staged：暂存状态，执行git commit则将修改同步到库中，这时库中的文件和本地文件又变为一致，文件为Unmodify状态，执行git reset HEAD filename取消暂存，文件状态为Modified

查看文件状态：git status /git status [filename]
添加所有文件到暂存区：git add .
提交暂存区的所哟文件到本地仓库：git commit -m

**忽略文件**：
如果希望某些文件不纳入版本控制，在目录下建立“.gitignore”，此文件有如下规则：
#为注释
*.txt    #忽略所有.txt结尾的文件
!lib.txt #但lib.txt除外
/temp #仅忽略项目根目录下的TODO文件，不包括其它目录temp
build/ #忽略build/目录下的所有文件
doc/*.txt #会忽略 doc/notes.txt 但不包括doc/server/arch.txt

### Git分支
列出所有本地分支：git branch
列出所有远程分支：git branch -r
新建分支，但依然停留在当前分支：git branch [branch-name]
新建分支，并切换到该分支：git checkout -b [branch]
合并指定分支到当前分支：git merge [branch]
删除分支：git branch -d [branch-name]
删除远程分支：
git push origin --delete [branch-name]
git branch -dr [remote/branch]

### Git项目到github上
```
    git init
    git add .
    git commit -m "first commit"
    git remote add origin [url]
    git push -u origin master
```
                                                                                                                                                          