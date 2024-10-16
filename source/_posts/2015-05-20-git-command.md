---
layout: post
title: Git常用命令和规范
categories: Git
description: Git常用命令和规范
index_img: 
date: 2015-05-20 09:09:09
tags: [Git]
---

### ssh key 配置

```Shell
git config  —global user.email xxxx@gmail.com
git config —global user.name zhangsan

ssh-keygen -t rasa -C “xxxx@gmail.com” 生成ssh key 
cat ~/.ssh/id_rsa.pub //生成后需要把sshkey填入gitlab中

git remote show origin //git 查看远程仓库信息
```

### git 检出仓库

```Shell
git clone git@server:app.git  workCode  // 检出仓库并命名为 workCode
git remote -v 			     //查看git跟踪的远程仓库地址
git remote add [name] [url]  //添加跟踪的远程仓库  [仓库名称] [仓库地址]
git remote rm [name]  	     //移除跟踪的仓库  还需要 git push origin master
```
### 常用基本操作

** git 检查
```Shell
git status  //查看变更的文件
git diff —stat //查看变更的文件
git diff	//查看详细变更内容
git diff test //查看test文件的变更
git diff HEAD 22bc77606de1d06bb589b316b9a7205cf42b7434 ./lib  //比较当前 lib目录 与 commit HEAD** 之间的差别
```

** git 日志
```Shell
git log //查看每一次的commit  内容
```

** git 提交
```Shell
git fetch +  git merge  == git pull 
git add .
git rm -r aa //git删除一个文件
git commit -m “fix: this is bug fix”
git push origin master //提交
```

** git 发布
```Shell
git tag  release/0.0.1
git push origin release/0.0.1
```

** git 代码合并
```Shell
git fetch origin master 				//从远程的origin的master主分支下载最新的版本到origin/master分支上
git log -p master  origin/master 		//比较本地的master分支和origin/master分支的差别
git merge origin/master  				//进行合并
//上述过程其实可以用以下更清晰的方式来进行：
git fetch origin master:tmp
git diff tmp 
git merge tmp
```
** git 分支操作
```Shell
git branch //列出所有分支
git branch daily/0.0.1  //创建分支
git checkout daily/0.0.1 //切换到daily/0.0.1 分支
git checkout -b daily/0.0.1 //创建并切换分支
git branch -d daily/0.0.1 //删除本地分支
git push origin —delete daily/0.0.1  //删除远程分支
```

** github个人主页代码托管：
```Shell
git checkout -b gh-pages    //新建分支并切换到分支”
git push -u origin gh-pages    //把文件推到分支”
```

** git commit日志规范
```Shell
feat: 新增功能。
fix: 修复 bug。
docs: 文档相关的改动。
style: 对代码的格式化改动，代码逻辑并未产生任何变化。
test: 新增或修改测试用例。
refactor: 重构代码或其他优化举措。
chore: 项目工程方面的改动，代码逻辑并未产生任何变化。
```

>例子： git commit -m "feat: 新增点击医生头像可跳转医生主页"

## github与gitlab通用
[参考文章](http://xuyuan923.github.io/2014/11/04/github-gitlab-ssh/);
