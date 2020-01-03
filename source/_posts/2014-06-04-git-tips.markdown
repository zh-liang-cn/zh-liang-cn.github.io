---
layout: post
title: Git笔记
date: 2014-06-04 14:56:26
category: Linux
excerpt: 工作中越来越离不开<em><a href="http://git-scm.com">git</a></em>，深感<em><a href="http://git-scm.com">git</a></em>的功能强大，基本上你会用到的、你能想到的功能都可以实现。本文是笔者遇到的问题以及解决方法的汇总，希望对读者有所帮助。
---

工作中越来越离不开 [git](http://git-scm.com/)，深感 [git](http://git-scm.com/) 的功能强大，基本上你会用到的、你能想到的功能都可以实现。本文是笔者遇到的问题以及解决方法的汇总，希望对读者有所帮助。

## git工具

* tig：控制台的GUI工具，通过`sudo apt-get install tig`或者`brew install tig`即可安装，强烈推荐
* gitk：GUI工具，使用tcl/tk编写，Ubuntu上默认已经安装了，直接在控制台执行`gitk`即可以启动
* SourceTree：著名的[Atlassian](https://www.atlassian.com)的_免费_产品，支持Mac和Windows，但是_不_支持Linux，下载地址：[sourcetreeapp.com](http://www.sourcetreeapp.com/)

## 只提交文件中的某几行改动而不是全部的改动

执行`git add -p <filename.java>`命令，git会逐块（hunk）地询问你是否要提交这一块。

```
Stage this hunk [y,n,q,a,d,/,j,J,g,s,e,?]? ??
y - stage this hunk
n - do not stage this hunk
q - quit; do not stage this hunk nor any of the remaining ones
a - stage this hunk and all later hunks in the file
d - do not stage this hunk nor any of the later hunks in the file
g - select a hunk to go to
/ - search for a hunk matching the given regex
j - leave this hunk undecided, see next undecided hunk
J - leave this hunk undecided, see next hunk
k - leave this hunk undecided, see previous undecided hunk
K - leave this hunk undecided, see previous hunk
s - split the current hunk into smaller hunks
e - manually edit the current hunk
? - print help
```

参考：

* [Commit only part of a file in Git](http://stackoverflow.com/questions/1085162/commit-only-part-of-a-file-in-git)

## Checkout远程分支


	$ git checkout --track origin/daves_branch


这会创建一个名称 daves_branch 的本地分支

参考：

* [http://stackoverflow.com/questions/9537392/git-fetch-remote-branch](http://stackoverflow.com/questions/9537392/git-fetch-remote-branch)

## 将已经commit到本地但是还没push到远程仓库的commit拆分成多个commit

如果是上一个commit，即HEAD\~1，可以`git reset HEAD~1 --soft`，本地仓库会回到上一个commit提交前的状态，然后根据你的需要分成多个commit即可。

如果是前几次commit，比如HEAD~3，可以按如下步骤：

* 使用`git rebase -i HEAD~4`进入交互式rebase
* 将你需要拆分的那个commit改成edit状态
* 此时当前工作的commit是处于已经提交的状态，使用`git reset HEAD~1`将其恢复成提交前的状态
* 根据你的需要分别使用`git add`和`git commit`提交

这里：

* 可以使用`git add -p <file name>`来实现对一个文件中修改分多次提交
* 可以使用`git commit -c ORIG_HEAD`来重用拆分的那个commit的commit message

参考：

* [http://stackoverflow.com/questions/1440050/how-to-split-last-commit-into-two-in-git](http://stackoverflow.com/questions/1440050/how-to-split-last-commit-into-two-in-git)
* [Git: How to split up a commit buried in history](http://stackoverflow.com/questions/4307095/git-how-to-split-up-a-commit-buried-in-history)

## 撤销reset/rebase/cherry-pick/checkout/commit操作

使用`git reset HEAD~1`可以很方便的撤销上一次的commit，但是仍然将修改过的文件保存在缓存区，有时会想再将这个commit还原回来，这时该怎么办呢？

Git实际上将HEAD的变动都保存在了reflog中，使用`git reflog`可以查看，你也可以认为git保存了所有的_commit_、_reset_、_rebase_、_cherry-pick_、_checkout_操作的历史记录，因此代码一旦commit就不用担心丢失了（除非删掉了本地库）。

使用`git reflog`可以看到所有这些操作，然后使用`git reset`

	$ git reflog -5
	c60995b HEAD@{0}: rebase -i (finish): returning to refs/heads/master
	c60995b HEAD@{1}: rebase -i (pick): add d
	91c3a01 HEAD@{2}: commit: add c
	9ac23cf HEAD@{3}: commit: add b
	38b72b4 HEAD@{4}: reset: moving to HEAD~1

如果需要回到`91c3a01 HEAD@{2}: commit: add c`这个commit，直接执行`git reset 91c3a01`或者更加直观`git reset HEAD@{2}`即可。

这里的`-5`意思是只显示5行，`git log -5`也只显示最近5个commit的log

参考

* [http://stackoverflow.com/questions/2510276/undoing-git-reset](Undoing git reset?)
* [reflog, your safety net](http://gitready.com/intermediate/2009/02/09/reflog-your-safety-net.html)

## `git co`与`git st`命令是怎么回事

git允许定义命令的别名，根据约定俗成，这里`co`是`checkout`的别名、`st`是`status`的别名，也就是说可以使用`git co`来替代`git checkout`命令。

有两种方法可以添加这种别名，第一种是直接编辑_~/.gitconfig_文件，如下

```
[alias]
  co = checkout
  ci = commit
  st = status
  br = branch
  hist = log --pretty=format:\"%h %ad | %s%d [%an]\" --graph --date=short
  type = cat-file -t
  dump = cat-file -p
```

第二种是通过`git config`命令来配置，依次执行：

```
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.br branch
git config --global alias.hist 'log --pretty=format:"%h %ad | %s%d [%an]" --graph --date=short'
git config --global alias.type 'cat-file -t'
git config --global alias.dump 'cat-file -p'
```

## 取出某个特定文件的某个特定版本

`git checkout`命令用来切换分支，会取出整个分支的代码，但是也可以用来取出某个特定的文件：

```
git checkout <commit SHA> <filepath>
```

## 设置commit message模板

将commit message写到一个文件中，然后执行：

```
git config --global commit.template ~/.git.commit.template
```

## 将commit从一个repo移动到另一个repo

使用_git format-patch_命令会为每一个commit生成一个patch文件，然后使用_git apply_命令将生成额patch文件应用到另一个仓库：

```
git format-patch <first commit SHA>~..<first commit SHA>
git apply <path to patch>
```

参考：

* [git-format-patch(1) Manual Page](https://www.kernel.org/pub/software/scm/git/docs/git-format-patch.html)
* [git-apply(1) Manual Page](https://www.kernel.org/pub/software/scm/git/docs/git-apply.html)
* [Git: Move a commit between repositories](http://makandracards.com/makandra/5531-git-move-a-commit-between-repositories)

## 忽略文件夹

只需要将文件夹名称加入_.gitignore_文件即可，注意，所有子文件夹下的这个目录都会被忽略，除非

```
config/                 # 忽略config/文件夹
!configuration/config/  # 不要忽略configuration文件夹下的config文件夹
```

参考：

* [gitignore(5) Manual Page](https://www.kernel.org/pub/software/scm/git/docs/gitignore.html)

## git stash 命令

`git stash`命令可以将改动暂存起来:

	$ git stash
	$ git stash pop
 	$ git stash list
	$ git stash drop

查看stash中的改动：

	$ git stash show
	$ git stash show -p
	$ git stash show -p stash @{1}

参考：

* [Git: see what's in a stash without applying stash](http://stackoverflow.com/questions/10725729/git-see-whats-in-a-stash-without-applying-stash)

## push本地的某个commit

如果临时做了一个patch，只希望push这一个commit到repo中:

	$ git push <remotename> <commit SHA>:<remotebranchname>

参考：

* [git - pushing specific commit](http://stackoverflow.com/questions/3230074/git-pushing-specific-commit)

## git远程分支操作

### push本地分支到repo

在本地创建了一个分支后，需要将这个分支push到远程repo中：

	$ git push origin new_feature

### 删除远程repo中的分支

	$ git push origin :new_feature # 注意里面的冒号

参考：

* [push and delete remote branches](http://gitready.com/beginner/2009/02/02/push-and-delete-branches.html)

### 将本地分支恢复到远程仓库的HEAD状态

	$ git fetch origin
	$ git reset --hard origin/mainline
	$ git clean -f -d

其中：_-f_表示_force_，_-d_表示_clean directory_

参考：

* [Reset and sync local respository with remote branch](http://ocpsoft.org/tutorials/git/reset-and-sync-local-respository-with-remote-branch/)

### 只merge另一个分支的某个commit

使用_git cherry-pick_命令：

	$ git cherry-pick <commit SHA>

参考：

* [How to merge a specific commit in git](http://stackoverflow.com/questions/881092/how-to-merge-a-specific-commit-in-git)

## 删除还未push的commit

_HEAD~1_表示恢复到上一个版本，这里可以替换成commit的SHA值：

	$ git reset --hard HEAD~1

## 修改还未push的上一次commit

如果需要对刚刚提交到本地repo的commit进行修改：

	$ git reset --soft HEAD^
	$ do some change
	$ git add -A
	$ git commit -c ORIG_HEAD

参考：

* [How to undo the last Git commit?](http://stackoverflow.com/questions/927358/how-to-undo-the-last-git-commit)

## 生成SSH public keys

	$ ssh-keygen -t rsa

参考：

* [Set Up Git](https://help.github.com/articles/set-up-git)

## 本地分支

### 显示本地分支信息

	$ git branch
	* master

### 创建新的branch

	$ git branch testing
	$ git branch
	* master
	  testing

### 切换到新创建的branch

	$ git checkout testing
	$ git branch
	  master
	* testing

### 创建branch的同时切换到新创建的branch

	$ git checkout -b testing2
	$ git branch
	  master
	  testing
	* testing2

### 删除branch

	$ git branch -d testing
	$ git branch
	  master
	* testing2

### 合并分支

	$ git branch master
	$ git branch
	* master
	  testing2
	$ git merge testing2


参考：

* [http://gitref.org/branching/](http://gitref.org/branching/)
* [http://darrinholst.com/post/359817782](http://darrinholst.com/post/359817782)

## 修改最近一次提交的message

使用_git commit_的_--amend_参数：

  git commit --amend -m "some other message"

修改前几次提交的但是还没push到remote repository的commit的message

	$ git lg
	* 6119426 - (master) third commit (3 seconds ago by Darrin Holst)
	* b6a0a9f - we want to change this message (23 seconds ago by Darrin Holst)
	* e24b57d - first commit (37 seconds ago by Darrin Holst)

## 交互式rebase

使用_git rebase_的_-i_参数：

	$ git rebase -i HEAD~2

将在编辑器中打开：

	pick b6a0a9f we want to change this message
	pick 6119426 third commit

	# Rebase e24b57d..6119426 onto e24b57d
	#
	# Commands:
	#  p, pick = use commit
	#  e, edit = use commit, but stop for amending
	#  s, squash = use commit, but meld into previous commit

将第一个commit改成edit：

	Change “pick” to “edit” for those that you want to edit and save that file

	edit b6a0a9f we want to change this message
	pick 6119426 third commit

	# Rebase e24b57d..6119426 onto e24b57d
	#
	# Commands:
	#  p, pick = use commit
	#  e, edit = use commit, but stop for amending
	#  s, squash = use commit, but meld into previous commit

进行相应的修改：

	Now amend that commit

	$ git commit --amend -m "some other message"

接着继续rebase操作：

	$ git rebase --continue
	$ git lg
	* cec3fba - (master) third commit (2 seconds ago by Darrin Holst)
	* c146f90 - some other message (29 seconds ago by Darrin Holst)
	* e24b57d - first commit (5 minutes ago by Darrin Holst)
