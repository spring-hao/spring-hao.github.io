---
title: git使用
date: 2021-08-25 09:53:44.684
updated: 2022-06-09 14:35:21.425
url: /archives/git使用
categories: 
- 工具
tags: 
- git
---



# git日常使用的小问题

## 1. 嵌套仓库

如果在一个git仓库中嵌套了一个git仓库，那么被嵌套的git仓库的改动，不能被大git仓库检测到。

**解决方案：**

- 可以使用submodule，当引入子仓库时，使用如下命令即可：

  ```
  git submodule add https://github.com/***
  ```

  作用类似git clone，但是他会在父仓库的下面新建.gitmodules文件，并且包含以下内容

  ```
  [submodule "apps/firstApp"]
  	path = apps/firstApp
  	url = https://github.com/muchang/mean-seed-app.git
  ```

  这一段表示子仓库的位置，以及子仓库的远程仓库的地址。

  删除子仓库并且commit之后，这个文件和这个子仓库有关的部分就会消失。

- 本质和第一个方案类似：

  ```
  git clone --recursive https://github.com/***
  ```

  **这两种方案可以同时维护两个仓库，字仓库也可以随时拉取更新**

- 删除字仓库`.git`目录，等于放弃了字仓库的远程关联

# git介绍

**Git是目前世界上最先进的分布式版本控制系统**,与常用的版本控制工具 CVS, Subversion 等不同，它采用了分布式版本库的方式，不必服务器端软件支持。

集中式vs分布式:[Git教程- 廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/896043488029600/896202780297248)
> - 集中式版本控制系统，版本库是集中存放在中央服务器的，而干活的时候，用的都是自己的电脑，所以要先从中央服务器取得最新的版本，然后开始干活，干完活了，再把自己的活推送给中央服务器。中央服务器就好比是一个图书馆，你要改一本书，必须先从图书馆借出来，然后回到家自己改，改完了，再放回图书馆。
> - 段落引用分布式版本控制系统根本没有“中央服务器”，每个人的电脑上都是一个完整的版本库，这样，你工作的时候，就不需要联网了，因为版本库就在你自己的电脑上。既然每个人电脑上都有一个完整的版本库，那多个人如何协作呢？比方说你在自己电脑上改了文件A，你的同事也在他的电脑上改了文件A，这时，你们俩之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。

Git 更像是把数据看作是对小型文件系统的一系列快照。 在 Git 中，每当你提交更新或保存项目状态时，它基本上就会对当时的全部文件创建一个快照并保存这个快照的索引。 为了效率，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。 Git 对待 数据更像是一个 快照流。++这是 Git 与几乎所有其它版本控制系统的重要区别。++ 
![存储项目随时间改变的快照.png](/upload/2022/02/%E5%AD%98%E5%82%A8%E9%A1%B9%E7%9B%AE%E9%9A%8F%E6%97%B6%E9%97%B4%E6%94%B9%E5%8F%98%E7%9A%84%E5%BF%AB%E7%85%A7-2494d6dda12142d58eef358e12fa39a8.png)

Git 有三种状态，你的文件可能处于其中之一: 已提交(committed)、已修改(modified) 和 已暂存(staged)。
- 已修改表示修改了文件，但还没保存到数据库中。
- 已暂存表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。 
- 已提交表示数据已经安全地保存在本地数据库中。

这会让Git项目拥有三个阶段:工作区、暂存区以及 Git 目录。
# 初次运行 Git 前的配置
Git 自带一个 git config 的工具来帮助设置控制 Git 外观和行为的配置变量。 这些变量存储在三个不同的位置:
1. /etc/gitconfig 文件: 包含系统上每一个用户及他们仓库的通用配置。 如果在执行 git config 时带上 --system 选项，那么它就会读写该文件中的配置变量。
2. ~/.gitconfig 或 ~/.config/git/config 文件:只针对当前用户。 你可以传递 --global 选项让 Git 读写此文件，这会对你系统上 所有 的仓库生效。
3. 当前使用仓库的 Git 目录中的 config 文件(即 .git/config):针对该仓库。 你可以传递 --local 选 项让 Git 强制读写此文件，虽然默认情况下用的就是它。。 (当然，你需要进入某个 Git 仓库中才能让该选项生效。)

每一个级别会覆盖上一级别的配置，所以 .git/config 的配置变量会覆盖 /etc/gitconfig 中的配置变量。
## 用户信息

安装完 Git 之后，要做的第一件事就是设置你的用户名和邮件地址。 这一点很重要，因为每一个 Git 提交都会使 用这些信息，它们会写入到你的每一次提交中，不可更改:
```bash
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```
## 检查配置信息

```bash
git config --list
```
# 创建本地仓库
## 1. 新建仓库

```bash
$ git init
Initialized empty Git repository in D:/testgit/.git/

//克隆一个仓库并命名为mylibgit
$ git clone https://github.com/libgit2/libgit2 mylibgit
```
## 2. 把新增文件添加仓库
```git add```告诉Git，把文件添加到仓库,```git add```后跟文件名或者```.```,```.```表示所有文件
```bash
git add readme.txt
git add .
```
## 3. 文件提交到仓库
```bash
$ git commit -m "test"
[master (root-commit) 60bffb4] test
 1 file changed, 2 insertions(+)
 create mode 100644 readme.txt
```
```git commit```命令，```-m```后面输入的是本次提交的说明，方便在历史记录里方便地找到改动记录。
```git commit```命令执行成功后会告诉你，1 file changed：1个文件被改动（新添加的readme.txt文件）；2 insertions：插入了两行内容（readme.txt有两行内容）。

提交时记录的是放在暂存区域的快照。 任何还未暂存文件的仍然保持已修改状态，可以在下次提交时 纳入版本管理。 每一次运行提交操作，都是对你项目作一次快照，以后可以回到这个状态，或者进行比较。
```bash
git checkout -- readme.txt
```
意思就是，把readme.txt文件在工作区的修改全部撤销，就是让这个文件回到最近一次git commit或git add时的状态。
## 4. 忽略文件
一般我们总会有些文件无需纳入 Git 的管理，也不希望它们总出现在未跟踪文件列表。 通常都是些自动生成的文 件，比如日志文件，或者编译过程中创建的临时文件等。 在这种情况下，我们可以创建一个名为 .gitignore 的文件，列出要忽略的文件的模式。 
```bash
$ cat .gitignore
# 忽略所有的 .a 文件 
*.a

# 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件 
!lib.a

# 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO 
/TODO

# 忽略任何目录下名为 build 的文件夹 
build/

# 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt 
doc/*.txt

# 忽略 doc/ 目录及其所有子目录下的 .pdf 文件 
doc/**/*.pdf

```

# 版本管理
修改readme.txt文件,增加一些内容,随后查看状态:
```bash
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   readme.txt

no changes added to commit (use "git add" and/or "git commit -a")
```
```git status```查看仓库当前状态,上面的命令输出告诉我们，readme.txt被修改过了，但还没有准备提交的修改。

```bash
$ git diff readme.txt
diff --git a/readme.txt b/readme.txt
index f0ec47f..ee2c1ea 100644
--- a/readme.txt
+++ b/readme.txt
@@ -1,2 +1,2 @@
-Git is a version control system.
+Git is a distributed version control system.
 Git is free software
\ No newline at end of file

```
查看修改后的不同之处.之后再次添加和提交即可
## 版本回退
```commit```相当于一个快照,误删了文件，还可以从最近的一个commit恢复。在Git中，我们用`git log`命令查看历史提交记录.
```bash
$ git log
commit 49b50fc00a1af29c9deb45e4eaf01cf168b1dc69 (HEAD -> master)
Author: a
Date:   Wed Aug 25 10:36:17 2021 +0800

    3

commit 76aaebb8ba0216719c4e4d9c2bd5c88d55692d8a
Author: a
Date:   Wed Aug 25 10:34:54 2021 +0800

    add

commit 60bffb4ec6f9804bdbe7d6d8dd235facea0fdc43
Author: a
Date:   Wed Aug 25 10:11:19 2021 +0800

    test
```
不传入任何参数的默认情况下，`git log`会按时间先后顺序列出所有的提交，最近的更新排在最上面。git log 有许多选项可以帮助你搜寻你所要找的提交：
```bash
//-p 或 --patch ，它会显示每次提交所引入的差异(按 补丁 的格式输出)
//-2 选项来只显示最近的两次提交
$ git log -p -2

//oneline 会将每个提交放在一行显示，在浏览大量的提交时非常有用。 
//另外还 有 short，full 和 fuller 选项
$ git log --pretty=oneline
49b50fc00a1af29c9deb45e4eaf01cf168b1dc69 (HEAD -> master) 3
76aaebb8ba0216719c4e4d9c2bd5c88d55692d8a add
60bffb4ec6f9804bdbe7d6d8dd235facea0fdc43 test
```
当`oneline`与另一个 log 选项`--graph`结合使用时尤其有用。 这个选项添加了一些 ASCII 字符串 来形象地展示你的分支、合并历史:
```bash
  $ git log --pretty=format:"%h %s" --graph
  * 2d3acf9 ignore errors from SIGCHLD on trap
  *  5e3ee11 Merge branch 'master' of git://github.com/dustin/grit
  |\
  | * 420eac9 Added a method for getting the current branch.
  * | 30e367c timeout code and tests
  * | 5a09431 add timeout protection to grit
  * | e1193f8 support for heads with slashes in them
  |/
  * d6016bc require time for xmlschema
  *  11d191e Merge branch 'defunkt' into local
```
![git log的常用选项.png](/upload/2022/02/git%20log%E7%9A%84%E5%B8%B8%E7%94%A8%E9%80%89%E9%A1%B9-2cab391a68a6413597bb43d422b5a01c.png)

在Git中，用`HEAD`表示当前版本，也就是最新的提交49b50f...，上一个版本就是HEAD^ ，往上100个版本写100个^比较容易数不过来，所以写成`HEAD~100`。
如果要把版本回退到上个版本即add这个版本,使用命令:
```bash
$ git reset --hard HEAD~
HEAD is now at 76aaebb add
```

回退版本后悔了,需要恢复到新版本`git reset --hard 版本号`,这时`git log`无法查看版本号,可以使用命令查看自己的所有操作记录:
```bash
$ git reflog
76aaebb (HEAD -> master) HEAD@{0}: reset: moving to HEAD~
49b50fc HEAD@{1}: reset: moving to 49b50
76aaebb (HEAD -> master) HEAD@{2}: reset: moving to HEAD~
49b50fc HEAD@{3}: commit: 3
76aaebb (HEAD -> master) HEAD@{4}: commit: add
60bffb4 HEAD@{5}: commit (initial): test
```
## 删除文件
在文件管理器中把没用的文件删了,这个时候，Git知道你删除了文件，因此，工作区和版本库就不一致了，git status命令会立刻告诉你哪些文件被删除了：
```bash
$ git status
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	deleted:    test.txt

no changes added to commit (use "git add" and/or "git commit -a")
```
现在有两个选择，一是确实要从版本库中删除该文件，那就用命令git rm删掉，并且git commit：
```bash
$ git rm test.txt
rm 'test.txt'

$ git commit -m "remove test.txt"
[master d46f35e] remove test.txt
 1 file changed, 1 deletion(-)
 delete mode 100644 test.txt
```
下次提交时，该文件就不再纳入版本管理了。 如果要删除之前修改过或已经放到暂存区的文件，则必须使用 强制删除选项 -f(译注:即 force 的首字母)。 这是一种安全特性，用于防止误删尚未添加到快照的数据，这 样的数据不能被 Git 恢复。

另一种情况是删错了，因为版本库里还有呢，所以可以很轻松地把误删的文件恢复到最新版本：
```bash
$ git checkout -- test.txt
```
`git checkout`其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以一键还原。

另外一种情况是，我们想把文件从 Git 仓库中删除(亦即从暂存区域移除)，但仍然希望保留在当前工作目录中。 换句话说，想让文件保留在磁盘，但是并不想让 Git 继续跟踪。 当你忘记添加 .gitignore 文件，不小心把一个很大的日志文件或一堆 .a 这样的编译生成文件添加到暂存区时，这一做法尤其有用。 为达到这一目的，使用 --cached 选项:
```bash
$ git rm --cached README
```
## 移动文件
不像其它的 VCS 系统，Git 并不显式跟踪文件移动操作。 如果在 Git 中重命名了某个文件，仓库中存储的元数 据并不会体现出这是一次改名操作。 要在 Git 中对文件改名，可以这么做:
```bash
git mv file_from file_to
```
# 远程仓库
## 1. 将本地仓库与远程仓库关联
`git remote add <shortname> <url> `添加一个新的远程 Git 仓库，同时指定一个方便 使用的简写:
```bash
git remote add origin git@github.com:spring-hao/git-study.git
```
添加后，远程库的名字就是origin。

## 2. 推送
```bash
git push <remote> <branch>
```
把本地库的内容推送到远程，用git push命令，实际上是把当前分支master推送到远程。

只有当你有所克隆服务器的写入权限，并且之前没有人推送过时，这条命令才能生效。 当你和其他人在同一时 间克隆，他们先推送到上游然后你再推送到上游，你的推送就会毫无疑问地被拒绝。 你必须先抓取他们的工作 并将其合并进你的工作后才能推送。

由于远程库是空的，我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。

## 3. 删除远程库
```bash
//查看已经配置的远程仓库服务器
git remote -v
origin  git@github.com:michaelliao/learn-git.git (fetch)
origin  git@github.com:michaelliao/learn-git.git (push)
```
**如果没有推送权限，就看不到push的地址。**
根据名字删除
```bash
git remote rm origin
```
## 4. 克隆远程库
```bash
$ git clone git@github.com:michaelliao/gitskills.git

$ git fetch <remote>
```
`fetch`会访问远程仓库，从中拉取所有你还没有的数据。 执行完成后，你将会拥有那个远程仓库中所有分支 的引用，可以随时合并或查看。

如果使用`clone`命令克隆了一个仓库，命令会自动将其添加为远程仓库并默认以 “origin” 为简写。 所以`git fetch origin`会抓取克隆(或上一次抓取)后新推送的所有工作。必须注意`git fetch`命令只会将数据下载到你的本地仓库——它并不会自动合并或修改你当前的工作。 

## 5. 打标签
Git 可以给仓库历史中的某一个提交打上标签，以示重要。在Git中列出已有的标签非常简单，只需要输入git tag(可带上可选的-l选项--list):
```bash
$ git tag
  v1.0
  v2.0
```

# 分支管理
使用分支意味着你可以把你的工作从开发主线上分离开来， 以免影响开发主线。 在很多版本控制系统中，这是一个略微低效的过程——常常需要完全创建一个源代码目录的 副本。对于大项目来说，这样的过程会耗费很多时间。

在进行提交操作时，Git会保存一个提交对象----该对象会包含一个指向暂存内容的指针。不仅仅是这样，该提交对象还包含了作者的姓名和邮箱、提交时输入的信息以及指向它的父对象的指针。 首次提交产生的提交对象没有父对象，普通提交操作产生的提交对象有一个父对象，而由多个分支合并产生的提交对象有多个父对象

Git 的分支，其实本质上仅仅是指向提交对象的可变指针。 Git 的默认分支名字是 master。 在多次提交操作之后，你其实已经有一个指向最后那个提交对象的 master 分支。 master 分支会在每次提交时自动向前移动。
```
查看分支：git branch

创建分支：git branch <name>

切换分支：git checkout <name>或者git switch <name>

创建+切换分支：git checkout -b <name>或者git switch -c <name>

合并某分支到当前分支：git merge <name>

删除分支：git branch -d <name>
```

> Git 的 master 分支并不是一个特殊分支。 它就跟其它分支完全没有区别。 之所以几乎每一 个仓库都有master分支，是因为git init命令默认创建它，并且大多数人都懒得去改动它。

## 1. 创建与合并分支
> HEAD严格来说不是指向提交，而是指向master，master才是指向提交的，所以，HEAD指向的就是当前分支。[详细介绍](https://www.liaoxuefeng.com/wiki/896043488029600/900003767775424)

创建dev分支，然后切换到dev分支：
```bash
$ git switch -c dev
Switched to a new branch 'dev'
```
git switch 命令加上-c参数表示创建并切换，相当于以下两条命令：
```bash
$ git branch dev
$ git switch dev
```
用git branch命令查看当前分支：
```bash
$ git branch
* dev
  master
```
git branch命令会列出所有分支，当前分支前面会标一个*号。

在dev分支上修改提交后返回master分支,发现之前提交的内容不见了,因为刚才的操作是在dev分支,而master并没有改变.
```bash
$ git checkout master
Switched to branch 'master'
```
把dev分支的工作成果合并到master分支上：
```bash
$ git merge dev
Updating 4fad4f3..199e6c3
Fast-forward
 4.txt     | 0
 git-study | 1 +
 2 files changed, 1 insertion(+)
 create mode 100644 4.txt
 create mode 160000 git-study
```
在合并的时候，你应该注意到了“快进(fast-forward)”这个词。当你试图合并两个分支时，如果顺着一个分支走下去能够到达另一个分支，那么 Git 在合并两者的时候，只会简单的将指针向前推进 (指针右移)，因为这种情况下的合并操作没有需要解决的分歧——这就叫做 “快进(fast-forward)”。

合并完成后，就可以放心地删除dev分支了：
```bash
$ git branch -d dev
Deleted branch dev (was b17d20e)
```

> 由于 Git 的分支实质上仅是包含所指对象校验和(长度为 40 的 SHA-1 值字符串)的文件，所以它的创建和销毁 都异常高效。 创建一个新分支就相当于往一个文件中写入 41 个字节。
这与过去大多数版本控制系统形成了鲜明的对比，它们在创建分支时，将所有的项目文件都复制一遍，并保存到一个特定的目录，所需时间的长短，完全取决于项目的规模。 而在 Git 中，任何规模的项目都能在瞬间创建新分支。同时，由于每次提交都会记录父对象，所以寻找恰当的合并基础(译注:即共同祖先)也是同样的简单和高效。 

## 2. 遇到冲突时的分支合并
如果你在两个不同的分支中，对同一个文件的同一个部分进行了不同的修改，Git 就没法干净的合并它们。
## 3. 分支开发工作流
![git工作流](/upload/2021/11/git%E5%B7%A5%E4%BD%9C%E6%B5%81-4e949255699145e2811f45967aa734d3.png)