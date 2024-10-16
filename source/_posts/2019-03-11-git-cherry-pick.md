---
layout: post
title: git cherry-pick 学习笔记
categories: Java
description: git cherry-pick
index_img: https://xn--6or.us.kg/api/?raw
date: 2019-03-11 09:09:09
tags: [git,cherry-pick]
---

`git cherry-pick`可以理解为”挑拣”提交，它会获取某一个分支的单笔提交，并作为一个新的提交引入到你当前分支上。 当我们需要在本地合入其他分支的提交时，如果我们不想对整个分支进行合并，而是只想将某一次提交合入到本地当前分支上，那么就要使用`git cherry-pick`了。
```
git cherry-pick [<options>] <commit-ish>...

常用options:
    --quit                退出当前的chery-pick序列
    --continue            继续当前的chery-pick序列
    --abort               取消当前的chery-pick序列，恢复当前分支
    -n, --no-commit       不自动提交
    -e, --edit            编辑提交信息
```
#### git cherry-pick commitid

在本地仓库中，有两个分支:branch1和branch2，我们先来查看各个分支的提交：
```
# 切换到branch2分支
$ git checkout branch2
Switched to branch 'branch2'
$ 
$ 
# 查看最近三次提交
$ git log --oneline -3
23d9422 [Description]:branch2 commit 3
2555c6e [Description]:branch2 commit 2
b82ba0f [Description]:branch2 commit 1
# 切换到branch1分支
$ git checkout branch1
Switched to branch 'branch1'
# 查看最近三次提交
$ git log --oneline -3
20fe2f9 commit second
c51adbe commit first
ae2bd14 commit 3th
```

现在，我想要将branch2分支上的第一次提交内容合入到branch1分支上，则可以使用`git cherry-pick`命令：
```
$ git cherry-pick 2555c6e
error: could not apply 2555c6e... [Description]:branch2 commit 2
hint: after resolving the conflicts, mark the corrected paths
hint: with 'git add <paths>' or 'git rm <paths>'
hint: and commit the result with 'git commit'
```

当`cherry-pick`时，没有成功自动提交，这说明存在冲突，因此首先需要解决冲突,解决冲突后需要`git commit`手动进行提交：
```
$ git commit 
[branch1 790f431] [Description]:branch2 commit 2
 date: Fri Jul 13 18:36:44 2018 +0800
 1 file changed, 1 insertion(+)
 create mode 100644 only-for-branch2.txt
```

branch2分支上的第二次提交成功合入到了branch1分支上。

以上就是`git cherry-pick`的基本用法，如果没有出现冲突，该命令将自动提交。

#### git cherry-pick -n
如果不想`git cherry-pick`自动进行提交，则加参数`-n`即可。比如将branch2分支上的第三次提交内容合入到branch1分支上：
```
$ git cherry-pick 23d9422
[branch1 2c67715] [Description]:branch2 commit 3
 date: Fri Jul 13 18:37:05 2018 +0800
 1 file changed, 1 insertion(+)
```

查看提交log,它自动合入了branch1分支：
```
$ git log --oneline -3  2c67715 \[Description\]:branch2 commit 3 f8bc5db \[Description\]:branch2 commit 2  20fe2f9 commit second
```

如果不想进行自动合入，则使用`git cherry-pick -n`：
```
# 回退上次提交，再此进行cherry-pick
$ git reset --hard HEAD~
HEAD is now at f8bc5db [Description]:branch2 commit 2
$ git cherry-pick -n 23d9422
$ git status
On branch branch1
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
    modified:   only-for-branch2.txt
```

这时通过`git status`查看，发现已将branch2的提交获取但是没有合入。

#### git cherry-pick -e
如果想要在`cherr-pick`后重新编辑提交信息，则使用`git cherry-pick -e`命令，比如我们还是要将branch2分支上的第三次提交内容合入到branch1分支上，但是需要修改提交信息：
```
$ git cherry-pick \-e  23d9422
 [Description]:branch2 commit 3
 #
 # It looks like you may be committing a cherry-pick.
 # If this is not correct, please remove the file
 #       .git/CHERRY_PICK_HEAD
 # and try again.
```

#### git cherry-pick –continue, –abort，–quit

当使用`git cherry-pick`发生冲突后,将会出现如下信息：
```
$ git cherry-pick 23d9422
error: could not apply 23d9422... [Description]:branch2 commit 3
hint: after resolving the conflicts, mark the corrected paths
hint: with 'git add <paths>' or 'git rm <paths>'
hint: and commit the result with 'git commit'
```

这时如果要继续`cherry-pick`，则首先需要解决冲突，通过`git add .`将文件标记为已解决，然后可以使用`git cherry-pick --continue`命令，继续进行`cherry-pick`操作。

如果要中断这次`cherry-pick`,则使用`git cherry-pick --quit`，这种情况下当前分支中未冲突的内容状态将为`modified`，

如果要取消这次`cherry-pick`,则使用`git cherry-pick --abort`，这种情况下当前分支恢复到`cherry-pick`前的状态，没有改变。

#### git cherry-pick < branchname >
如果在`git cherry-pick`后加一个分支名，则表示将该分支顶端提交进`cherry-pick`，如：
```
$ git cherry-pick master
```
#### git cherry-pick ..
#### git cherry-pick ^HEAD 
以上两个命令作用相同，表示应用所有提交引入的更改，这些提交是branchname的祖先但不是HEAD的祖先，比如，现在我的仓库中有三个分支，其提交历史如下图：
```
               C<---D<---E  branch2
              /
master   A<---B  
              \
               F<---G<---H  branch3
                         |
                         HEAD
```

如果我使用`git cherry-pick ..branch2`或者`git cherry-pick ^HEAD branch2`,那么会将属于branch2的祖先但不属于branch3的祖先的所有提交引入到当前分支branch3上，并生成新的提交，执行命令如下:
```
$ git cherry-pick ..branch2
[branch3 c95d8b0] [Description]:branch2  add only-for-branch2
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 only-for-branch2
[branch3 7199a67] [Description]:branch2 modify for only-for-branch2--1
 1 file changed, 1 insertion(+)
[branch3 eb8ab62] [Description]:branch2 modify for only-for-branch2--2
 1 file changed, 1 insertion(+)
```
执行后的提交历史如下：
```

               C<---D<---E  branch2
              /
master   A<---B  
              \
               F<---G<---H<---C'<---D'<---E'  branch3
                                          |
                                         HEAD
```

