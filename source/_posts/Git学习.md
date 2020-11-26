---
title: Git学习
date: 2019-08-28 22:31:34
tags: [Git]
categories: Git
---
<meta name="referrer" content="no-referrer" />

# 创建版本库

版本库又名仓库，英文名repository，你可以简单理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。

创建版本库非常简单，创建一个空目录
```
mkdir learngit
cd learngit
```
之后，通过git init命令把这个目录变成Git可以管理的仓库：
```
git init
```
可以发现当前目录下多了一个.git的目录，这个目录是Git来跟踪管理版本库的，没事千万不要手动修改这个目录里面的文件，不然改乱了，就把Git仓库给破坏了。

* 如果你没有看到.git目录，那是因为这个目录默认是隐藏的，用ls -ah命令就可以看见。



# 把文件添加到版本库

使用命令`git add`告诉git,把文件添加到仓库
```
git add readme.txt
```

之后，使用命令`git commit`告诉Git，把文件提交到仓库
```
git commit -m "wrote a readme file"
```
简单解释一下git commit命令，-m后面输入的是本次提交的说明，可以输入任意内容，当然最好是有意义的，这样你就能从历史记录里方便地找到改动记录。

为什么Git添加文件需要add，commit一共两步呢？因为commit可以一次提交很多文件，所以你可以多次add不同的文件，比如
```
git add file1.txt
git add file2.txt file3.txt
git commit -m "add 3 files."
```


## 工作区和暂存区
对于上面的命令的深入理解，我们需要介绍一下工作区和暂存区

### 工作区（Working Directory）
就是你在电脑里能看到的目录，比如learngit文件夹就是一个工作区

### 版本库（Repository）
工作区有一个隐藏目录`.git`，这个不算工作区，而是Git的版本库。

### 暂存区（stage）
Git的版本库里存了很多东西，其中最重要的就是称为stage（或者叫index）的暂存区，还有Git为我们自动创建的第一个分支`master`，以及指向`master`的一个指针叫`HEAD`。

![](http://ww1.sinaimg.cn/large/006eDJDNly1g67191721tj30kh0argo6.jpg)

从上面可以知道，把文件往Git版本库里添加的时候，是分两步执行的
* 第一步用`git add`把文件添加进去，实际上就是把文件修改添加到暂存区；
* 第二步是用git commit提交更改，实际上就是把暂存区的所有内容提交到当前分支。

需要提交的文件修改通通放到暂存区，然后，一次性提交暂存区的所有修改。



# 版本回退

使用`git log`命令可以查看从最近到最远的提交日志。

如果嫌输出信息太多，可以试试加上`--pretty=oneline`参数

从日志中可以看到一大串类似`1094adb...`·的是`commit id`（版本号），这是由一个SHA1计算出来的一个非常大的数字，用十六进制表示

回退的方式也很简单

首先，Git需要知道当前版本是哪个版本，在Git中，用HEAD表示当前版本。上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个^比较容易数不过来，所以写成`HEAD~100`。

所以，回退版本使用命令`git reset`
```
git reset --hard HEAD^
```

回退到上一个版本后，如果还想再回去，比较困难，但是还是有办法的，只要上面的命令行窗口还没有被关掉，你就可以顺着往上找啊找啊，找到你想回去的版本的`commit id`，输入命令：
```
git reset --hard commit id
```
Git的版本回退速度非常快，因为Git在内部有个指向当前版本的HEAD指针，当你回退版本的时候，Git仅仅是把HEAD从指向另一个版本。然后顺便把工作区的文件更新了。所以你让HEAD指向哪个版本号，你就把当前版本定位在哪。

最坏的情况，假如你回退到了某个版本，关掉了电脑，第二天早上就后悔了，想恢复到新版本怎么办？找不到新版本的`commit id`怎么办？

在Git中，总是有后悔药可以吃的。使用`git reflog`。这个命令用来记录你的每一次命令，因此，使用这个命令之后，就可以找到你最新版本的`commit id`了



# 查看工作区和版本库里面最新版本的区别

比如查看工作区和版本库最新版本的xxx.txt的文件的区别，使用下面的命令
```
git diff HEAD -- xxx.txt
```


# 撤销修改

## 当只是在工作区修改，还没有add到暂存区时

你可以首先使用`git status`，可以发现，提示我们在工作区有文件修改。其中，有两句话，
```
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
```

第二句话就是说明 可以使用`git checkout -- <file>`来抛弃工作区修改的部分

命令`git checkout -- readme.txt`意思就是，把`readme.txt`文件在工作区的修改全部撤销，这里有两种情况：

* 一种是`readme.txt`自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；
* 一种是`readme.txt`已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

总之，就是让这个文件回到最近一次`git commit`或`git add`时的状态。

`git checkout -- file`命令中的`--`很重要，没有`--`，就变成了“切换到另一个分支”的命令


## 当你add到暂存区，还没有commit时
同样，使用`git status`查看， 发现修改只是添加到了暂存区，还没有提交，提示了
```
use "git reset HEAD <file>..." to unstage
```
Git告诉我们，可以使用命令`git reset HEAD <file>`把暂存区的修改撤销掉（unstage），重新放回工作区

`git reset`命令既可以回退版本，也可以把暂存区的修改回退到工作区。当我们用`HEAD`时，表示最新的版本

这时候，只是回退到了工作区，如果你还想丢弃工作区的修改，参考上一小节

## 当commit后
这时候，可以使用版本回退，参考前面所讲的版本回退内容


# 删除文件
在git中，删除也是一个修改操作

当我们add一个文件并commit后，我们在工作区删除掉这个文件通常使用的命令是`rm fileName`。这时候，Git知道你删除了文件，因此，工作区和版本库就不一致了，`git status`命令会立刻告诉你哪些文件被删除了：
```
$ git status
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	deleted:    test.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

如果你真的想从版本库中删除文件，则可以使用`git rm`删掉，并且`git commit`

如果你是手误删错的话，则可以使用下面的命令把误删的文件恢复到最新版本
```
git checkout
```

`git checkout`其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。



# 添加远程仓库

* 首先，登陆GitHub，然后，在右上角找到“Create a new repo”按钮，创建一个新的仓库
* 在Repository name填入项目名称，其他保持默认设置，点击“Create repository”按钮，就成功地创建了一个新的Git仓库

创建成功后，该github仓库是空的。GitHub告诉我们，可以从这个仓库克隆出新的仓库，也可以把一个已有的本地仓库与之关联，然后把本地仓库的内容推送到GitHub仓库。


如果我们有本地仓库了，则根据GitHub的提示，在本地仓库下运行命令：
```
git remote add origin git@github.com:你的github账户名/仓库名.git
```
> 添加后，远程库的名字就是origin，这是Git默认的叫法，也可以改成别的，但是origin这个名字一看就知道是远程库。

之后就可以把本地仓库的所有内容推送到远程库中
```
git push -u origin master

```
把本地库的内容推送到远程，用`git push`命令，实际上是把当前分支`master`推送到远程。

由于远程库是空的，我们第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令。

从现在起，只要本地作了提交，就可以通过命令：
```
git push origin master
```
把本地`master`分支的最新修改推送至GitHub

# 从远程库克隆

现在，假设我们从零开发，那么最好的方式是先创建远程库，然后，从远程库克隆。

我们假设已经创建好远程仓库，则使用命令`git clone`克隆一个本地库
```
git clone git@github.com:你的github账户名/仓库名称.git
```

Git支持多种协议，包括`https`，但通过`ssh`支持的原生`git`协议速度最快。


# 创建和合并分支

在默认一条分支的时候，Git通常将这条分支叫做`master`分支，`HEAD`严格来说不是指向提交，而是指向`master`，`master`才是指向提交的，所以，`HEAD`指向的就是当前分支。

一开始的时候，`master`分支是一条线，Git用`master`指向最新的提交，再用`HEAD`指向`master`，就能确定当前分支，以及当前分支的提交点：

![](http://ww1.sinaimg.cn/large/006eDJDNly1g6dequsw09j30ln09e0t4.jpg)

每次提交，`master`分支都会向前移动一步，这样，随着你不断提交，`master`分支的线也越来越长。

当我们创建新的分支，例如`dev`时，Git新建了一个指针叫`dev`，指向`master`相同的提交，再把`HEAD`指向`dev`，就表示当前分支在`dev`上：

```
创建dev分支，并切换到dev分支上
git checkout -b dev
# 这里git checkout命令加上-b参数表示创建并切换，相当于下面两条命令：
git branch dev
git checkout dev
```

![](http://ww1.sinaimg.cn/large/006eDJDNly1g6desio31mj30n80dhdgf.jpg)

Git创建一个分支很快，因为除了增加一个dev指针，改改HEAD的指向，工作区的文件都没有任何变化！


可以使用`git branch`命令查看当前分支
`git branch`命令会列出所有分支，当前分支前面会标一个`*`号。


从现在开始，对工作区的修改和提交就是针对`dev`分支了，比如新提交一次后，`dev`指针往前移动一步，而`master`指针不变：

![](http://ww1.sinaimg.cn/large/006eDJDNly1g6detptnx9j30u30d2wf5.jpg)

假如我们在`dev`上的工作完成了，就可以把`dev`合并到`master`上。具体做法就是直接把`master`指向`dev`的当前提交，就完成了合并：

* 首先切回到master分支
```
git checkout master
```
* 然后把`dev`分支的工作成果合并到master分支上：
```
git merge dev
```
`git merge`命令用于合并指定分支到当前分支。注意到输出的信息中的Fast-forward信息，Git告诉我们，这次合并是“快进模式”，也就是直接把master指向dev的当前提交，所以合并速度非常快。当然，也不是每次合并都能Fast-forward

![](http://ww1.sinaimg.cn/large/006eDJDNly1g6dev8izvej30pv0d7dgh.jpg)

合并完分支后，甚至可以删除`dev`分支。删除`dev`分支就是把`dev`指针给删掉，删掉后，我们就剩下了一条`master`分支：

```
git branch -d dev
```
![](http://ww1.sinaimg.cn/large/006eDJDNly1g6dew4geplj30nx09nmxk.jpg)


# 解决冲突

有时候，当我们合并分支的时候，会出现冲突，在冲突的文件里，Git会用`<<<<<<<`，`=======`，`>>>>>>>`标记出不同分支的内容

必须手动修改冲突后再提交。

使用带参数的`git log`也可以看到分支合并的情况
```
git log --graph --pretty=oneline --abbrev-commit
```

合并分支后，可以选择删除分支
```
git branch -d 分支名称
```
> 当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。解决冲突就是把Git合并失败的文件手动编辑为我们希望的内容，再提交。用git log --graph命令可以看到分支合并图。



# 分支管理策略
通常，合并分支时，如果可能，Git会用`Fast forward`模式，但这种模式下，删除分支后，会丢掉分支信息。

如果要强制禁用`Fast forward`模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息。

具体的命令如下,在合并的时候使用
```
git merge --no-ff -m "commit的内容" dev
```
`--no-ff`参数，表示禁用`Fast forward`。
因为本次合并要创建一个新的commit，所以加上-m参数，把commit描述写进去

可以看到，不使用`Fast forward`模式，`merge`后就像这样：

![](http://ww1.sinaimg.cn/large/006eDJDNly1g6dfx0z2myj30t80exwff.jpg)

在实际开发中，我们应该按照几个基本原则进行分支管理：
* `master`分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；
* 那在哪干活呢？干活都在dev分支上，也就是说，dev分支是不稳定的，到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本；
* 你和你的小伙伴们每个人都在dev分支上干活，每个人都有自己的分支，时不时地往dev分支上合并就可以了。

![](http://ww1.sinaimg.cn/large/006eDJDNly1g6dfz5a4fzj30v90883zo.jpg)


# Bug分支

当你遇到一个bug时，需要去修复，很自然地，会想到通过创建一个bug分支去修复它，但是这时候你正在`dev`分支上进行的工作还没有提交，并不是你不想提交，而是工作只进行到一半，还没法提交。这时候，可以使用git提供的`stash`功能，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作。
```
git stash
```
现在，用`git status`查看工作区，就是干净的.

首先确定要在哪个分支上修复bug，假定需要在`master`分支上修复，就从`master`创建临时分支

当修复完之后，切换到`master`分支，并完成合并，然后删除临时bug分支

修复完成bug后，我们回到`dev`分支，发现工作区是干净的，这时候使用`git stash list`命令查看，

工作现场还在，Git把stash内容存在某个地方了，但是需要恢复一下，有两个办法：
* 一是用`git stash apply`恢复，但是恢复后，stash内容并不删除，你需要用`git stash drop`来删除；
* 另一种方式是用git stash pop，恢复的同时把stash内容也删了：

使用上面的任意一个后，就恢复了现场，再次使用`git stash list`，就看不到任何stash的内容了

你可以多次stash，恢复的时候，先用`git stash list`查看，然后恢复指定的stash，用命令：
```
git stash apply stash@{0}
```

在master分支上修复了bug后，我们要想一想，dev分支是早期从master分支分出来的，所以，这个bug其实在当前dev分支上也存在。

同样的bug，要在dev上修复，我们可以使用Git提供的`cherry-pick`命令,让我们能复制一个特定的提交(也就是修改bug的提交)到当前分支
```
git cherry-pick commit_id
```



# Feature分支

每添加一个新功能，最好新建一个feature分支，在上面开发，完成后，合并，最后，删除该feature分支。

如果要丢弃一个没有被合并过的分支，可以通过`git branch -D <name>`强行删除。


# 多人协作

当从远程仓库克隆时，Git自动把本地的`master`分支和远程的`master`分支对应起来了，并且，远程仓库的默认名称是`origin`

查看远程库的信息，使用
 ```
 git remote
 ```
或者加上 -v显示更加详细的信息
```
git remote -v
```
上面显示了可以抓取和推送的`origin`的地址。如果没有推送权限，就看不到push的地址。


## 推送分支

推送分支，就是把该分支上的所有本地提交推送到远程库。推送时，要指定本地分支，这样，Git就会把该分支推送到远程库对应的远程分支上：

比如：
```
git push origin master
```
这个`master`指的是本地的分支

如果要推送其他分支的话，也就是
```
git push origin dev
```

但是，并不是一定要把本地分支往远程推送，那么，哪些分支需要推送，哪些不需要呢？
* `master`分支是主分支，因此要时刻与远程同步
* `dev`分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步；
* `bug`分支只用于在本地修复bug，就没必要推到远程了，除非老板要看看你每周到底修复了几个bug；
* `feature`分支是否推到远程，取决于你是否和你的小伙伴合作在上面开发。


## 抓取分支

可以使用clone命令
```
git clone github地址
```
默认情况下，只能看到本地的`master`，如果你想在`dev`分支上开发的话，则必须创建远程origin的dev分支到本地，使用下面的命令创建本地dev分支：
```
git checkout -b dev origin/dev
```

多人工作模式下在推送的时候可能会发生冲突，通常是这样来处理的
* 首先，可以试图用`git push origin <branch-name>`推送自己的修改；
* 如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并;如果`git pull`提示no tracking information，则说明本地分支和远程分支的链接关系没有创建，用命令`git branch --set-upstream-to <branch-name>   origin/<branch-name>`
* 如果合并有冲突，则解决冲突，解决的方法和分支管理中的解决冲突完全一样,解决后，提交，再push：

在本地创建和远程分支对应的分支，使用`git checkout -b branch-name origin/branch-name`，本地和远程分支的名称最好一致；

建立本地分支和远程分支的关联，使用`git branch --set-upstream    branch-name   origin/branch-name`；


# 创建标签
标签就是tag，tag就是一个让人容易记住的有意义的名字，它跟某个commit绑在一起

使用下面的命令就可以打标签
```
git tag v1.0
```
默认标签是打在最新提交的commit上的.

如果想对历史的某个commit打标签的话，只需要找到那个commit就可
```
git log --pretty=oneline --abbrev-commit
```

然后打上标签
```
git tag v0.9 commit_id
```

使用
```
git tag
```
可以查看tag

可以用`git show <tagname>`查看标签信息


还可以创建带有说明的标签，用`-a`指定标签名，`-m`指定说明文字：
```
git tag -a 标签名 -m "说明文字" commit_id
```

如果想要删除标签，使用
```
git tag -d tagname
```

因为创建的标签都只存储在本地，不会自动推送到远程。所以，打错的标签可以在本地安全删除

如果要推送某个标签到远程,使用命令
```
git push origin <tagname>
```

或者，一次性推送全部尚未推送到远程的本地标签：

```
git push origin --tags
```

如果标签已经推送到远程，要删除远程标签就麻烦一点，先从本地删除：
```
git tag -d tagname
```

从远程删除。删除命令也是push,但是格式如下：
```
git push origin :refs/tags/<tagname>
```

