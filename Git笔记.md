# 1  Git笔记

## 1.1  Git介绍

Git 是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。

Git 与常用的版本控制工具 CVS, Subversion 等不同，它采用了分布式版本库的方式，不必服务器端软件支持。

 

## 1.2  集中式vs分布式

### 1.2.1  集中式

![image-20200703223129699](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200703223129699.png)

<center>图 1集中式</center>

集中式版本控制系统，版本库是集中存放在中央服务器的，而干活的时候，用的都是自己的电脑，所以要先从中央服务器取得最新的版本，然后开始干活，干完活了，再把自己的活推送给中央服务器。中央服务器就好比是一个图书馆，你要改一本书，必须先从图书馆借出来，然后回到家自己改，改完了，再放回图书馆。

缺点：

1.  集中式版本控制系统最大的毛病就是必须联网才能工作，如果在局域网内还好，带宽够大，速度够快，可如果在互联网上，遇到网速慢的话，可能提交一个10M的文件就需要5分钟
2. 集中式版本控制系统的中央服务器要是出了问题，所有人都没法干活

### 1.2.2  分布式

 ![image-20200703223112397](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200703223112397.png)

<center>图 2分布式</center>

分布式版本控制系统根本没有“中央服务器”，每个人的电脑上都是一个完整的版本库，工作的时候，就不需要联网了，因为版本库就在你自己的电脑上。

既然每个人电脑上都有一个完整的版本库，那多个人如何协作呢？比方说你在自己电脑上改了文件A，你的同事也在他的电脑上改了文件A，这时，你们俩之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。

优点

1. 分布式版本控制系统通常也有一台充当“中央服务器”的电脑，但这个服务器的作用仅仅是用来方便“交换”大家的修改，没有它大家也一样干活，只是交换修改不方便而已。

 

## 1.3  Git命令

### 1.3.1  把当前目录变成Git管理的仓库

```
git init
```

### 1.3.2  将文件添加到仓库

```
git add <filename>
```

### 1.3.3  将文件提交到仓库

```
git commit -m "wrote a readme file"
```

-m后面填写提交的信息。

### 1.3.4  查看当前仓库状态

```
git status
```

### 1.3.5  展示同一文件的不同

```
git diff <filename> 
```

### 1.3.6  查看提交（commit）日志

```
git log
```

### 1.3.7  版本回溯

首先，Git必须知道当前版本是哪个版本，在Git中，用HEAD表示当前版本，上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100。

```
git reset --hard HEAD^
git reset --hard 1094a
根据版本号
```

 ![image-20200703223936494](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200703223936494.png)

<center>图 3</center>

commit后面跟的是提交的版本号。

### 1.3.8  查看操作记录

```
git reflog
```

### 1.3.9  撤销文件的修改

```
git checkout -- <filename>
```

命令git checkout -- readme.txt意思就是，把readme.txt文件在工作区的修改全部撤销，这里有两种情况：

l 一种是readme.txt自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；

l 一种是readme.txt已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

```
git reset HEAD <filename>
```

用HEAD时，表示回退到最新的版本。

### 1.3.10     删除文件

```
git rm <filename>
git commit -m "remove test.txt"
```

### 1.3.11     连接远程仓库（SSH模式）

```
git remote add origin git@github.com:michaelliao/learngit.git
```

### 1.3.12     将本地内容推送到远程库

```
git push -u origin master:master
```

第一个master是本地分支名，第二个master是远程分支名。

第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。

```
git push origin master
```

### 1.3.13     克隆远程项目

```
git clone git@github.com:michaelliao/gitskills.git
```

### 1.3.14     分支

#### 1.3.14.1   本地创建dev分支，并换到dev分支 ，同时远程也创建dev分支，并提交信息

```
git checkout -b dev
```

git checkout命令加上-b参数表示创建并切换。

以下指令分别表示创建和切换

```
$ git branch dev
$ git checkout dev
```

本地分支推送到远程服务器时，远程分支自动创建，推送本地分支到远程

```
git push --set-upstream <remote-name> <local-branch-name>:<remote-branch-name>
```

#### 1.3.14.2   查看所有分支

```
git branch -a
```

#### 1.3.14.3   dev分支合并到master

```
git merge dev
```

git merge命令用于合并指定分支到当前分支。合并后，再查看readme.txt的内容，就可以看到，和dev分支的最新提交是完全一样的。

#### 1.3.14.4   删除分支

```
git branch -d dev
```

#### 1.3.14.5  本地创建并切换分支， 与checkout作用相同

```
git switch -c dev
```

**切换**

```
git switch master
```

本地分支推送到远程服务器时，远程分支自动创建，推送本地分支到远程

```
git push --set-upstream <remote-name> <local-branch-name>:<remote-branch-name>
```

#### 1.3.14.6   查看分支合并情况

```
git log --graph --pretty=oneline --abbrev-commit
```

#### 1.3.14.7   强行删除未被merge分支

```
git branch -D feature-vulcan
```

需要使用大写的-D参数

#### 1.3.14.8   查看远程仓库名称及信息

```
git remote
git remote -v
```

#### 1.3.14.9   推送分支

```
//将本地master分支推送到远程origin分支上
git push origin master
//将本地dev分支推送到远程origin分支上
git push origin dev
```

#### 1.3.14.10  抓取分支

抓取远程的分支，将远程的origin/dev抓取下来，并且放入本地的dev中：

```
git checkout -b dev origin/dev
```

#### 1.3.14.11  rebase操作可以把本地未push的分叉提交历史整理成直线

```
git rebase
```

### 1.3.15     标签管理

#### 1.3.15.1   创建标签

```
git tag v1.0
```

#### 1.3.15.2   查看所有标签

```
git tag
```

#### 1.3.15.3   对历史提交创建标签

找到历史提交的commit id

```
git log --pretty=oneline --abbrev-commit
```

根据commit id创建标签

```
git tag v0.9 f52c633
```

#### 1.3.15.4   查看标签信息

```
git show v0.9
```

#### 1.3.15.5   删除标签

```
git tag -d v0.1
```

创建的标签都只存储在本地，不会自动推送到远程。所以，打错的标签可以在本地安全删除。

#### 1.3.15.6   将标签推送到远程

```
//推送v1.0
git push origin v1.0
//推送所有未推送的标签
git push origin --tags
```

#### 1.3.15.7   删除远程标签

如果标签已经推送到远程，要删除远程标签就麻烦一点，先从本地删除：

```
git tag -d v0.9
```

然后，从远程删除。删除命令也是push，但是格式如下：

```
git push origin :refs/tags/v0.9
```

  

## 1.4  工作区和暂存区

### 1.4.1  工作区（Working Directory）

就是在电脑里能看到的目录，比如GitRepository文件夹就是一个工作区：

 ![image-20200703224009134](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200703224009134.png)

<center>图 4</center>

 

## 1.5  版本库

### 1.5.1  什么是版本库？

版本库又名仓库，英文名repository，你可以简单理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。

 

### 1.5.2  版本库（Repository）

工作区有一个隐藏目录.git，这个不算工作区，而是Git的版本库。

Git的版本库里存了很多东西，其中最重要的就是称为stage（或者叫index）的暂存区，还有Git为我们自动创建的第一个分支master，以及指向master的一个指针叫HEAD。

**之前执行的操作**

第一步是用git add把文件添加进去，实际上就是把文件修改添加到暂存区；

第二步是用git commit提交更改，实际上就是把暂存区的所有内容提交到当前分支

 

## 1.6  分支

### 1.6.1  解决分支冲突

#### 1.6.1.1 不同分支合并

如果分支a的readme.txt内容被修改，并且被提交，同时，分支b的readme.txt内容也被修改，并且被提交，在a分支中执行

```
git merge b
```

会提示出现冲突

```
Auto-merging readme.txt
CONFLICT (content): Merge conflict in readme.txt
Automatic merge failed; fix conflicts and then commit the result.
```

为了解决分支冲突，我们需要在a分支中手动修改文件，将文件内容合并到a中的readme.txt文件中，再进行提交，就能解决冲突。

#### 1.6.1.2 相同分支提交

如果甲和乙对相同分支的相同文件进行更改，甲先提交，乙再提交就会出现冲突，乙需要将甲提交的信息抓取下来，用git pull把最新的提交从origin/dev抓下来，然后，在本地合并，解决冲突，再推送。

如果出现git pull失败，原因是没有指定本地dev分支与远程origin/dev分支的链接，根据提示，设置dev和origin/dev的链接：

```
git branch --set-upstream-to=origin/dev dev
```

再次pull，merge之后提交即可。

 

### 1.6.2  分支策略

![image-20200703233931911](https://raw.githubusercontent.com/kender1314/NotePicture/master/image-20200703233931911.png)

<center>图 5</center>

在实际开发中，我们应该按照几个基本原则进行分支管理：

l 首先，master分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活。

l 干活都在dev分支上，也就是说，dev分支是不稳定的，到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本；

l 每个人都在dev分支上干活，每个人都有自己的分支，时不时地往dev分支上合并就可以了。

### 1.6.3  Bug分支

软件开发中，有了bug就需要修复，在Git中，每个bug都可以通过一个新的临时分支来修复，修复后，合并分支，然后将临时分支删除。当你接到一个修复一个代号101的bug的任务时，可以创建一个分支issue-101来修复它。

如果当前的分支的工作还未提交，并且因为某些原因，无法提交。遇到这种情况，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作：

```
git stash
```

首先确定要在哪个分支上修复bug，假定需要在master分支上修复，就从master创建临时分支：

```
$ git checkout master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 6 commits.
  (use "git push" to publish your local commits)
 
$ git checkout -b issue-101
Switched to a new branch 'issue-101'
```

修复完成后，切换到master分支，并完成合并，最后删除issue-101分支：

```
$ git switch master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 6 commits.
  (use "git push" to publish your local commits)
 
$ git merge --no-ff -m "merged bug fix 101" issue-101
Merge made by the 'recursive' strategy.
 readme.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

*注：通常，合并分支时，如果可能，**Git**会用**Fast forward**模式，但这种模式下，删除分支后，会丢掉分支信息。可以通过**--no-ff**参数，表示禁用**Fast forward**。*

修改完成之后，返回之前的工作分区，获取存储的工作状态：

```
git stash list
```

恢复之前的工作状态，并且删除之前存储的工作状态：

```
git stash pop
```

如果git stash list获取到多个存储的工作状态，可以用git stash apply恢复指定的工作状态，但是恢复后，stash内容并不删除，你需要用git stash drop来删除；

```
git stash apply stash@{0}
git stash drop
```

### 1.6.4  如何在多个分支同步修复的bug

同样的bug，要在dev上修复，我们只需要把4c805e2 fix bug 101这个提交所做的修改“复制”到dev分支。注意：我们只想复制4c805e2 fix bug 101这个提交所做的修改，并不是把整个master分支merge过来。

为了方便操作，Git专门提供了一个cherry-pick命令，让我们能复制一个特定的提交到当前分支：

```
$ git branch
* dev
  master
$ git cherry-pick 4c805e2
```

 

 

 

 

 

 

 

 