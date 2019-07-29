---
title: GitCommand
date: 2019-07-22 16:41:45
tags: 
 -git
 -VersionControl
categories:
 -git
---

# git分布式版本控制工具

##### 创建版本库: 

​	`$  mk dir folder_name`	  	  <!--创建版本库文件夹-->

​	`$  cd folder_name` 			 <!--进入当前文件夹-->

​	`$  pwd`				   	<!--查看当前路径-->

###### `$  git init`					<!--将当前目录变成git可管理的仓库-->

##### 添加文件后将文件添加到受git管理的文件

​	`$  git add file_name` 	  	   		      <!--文件添加到仓库-->

​	`$  git commit  -m "discribe"`		    	      <!--文件提交到仓库-->

##### 修改文件后查看版本库状态/文件修改查看

​	`$ git status`					<!--查看仓库当前状态-->

​	`$  git diff`					    <!--查看文件修改内容-->

##### 确认修改内容后重复add-commit文件提交版本库	

​		`git status`命令掌握工作区的状态。

​		如果`git status`告诉你有文件被修改过，用`git diff`可以查看修改内容。

### 二 版本控制及文件操作

##### 1.版本回退

​		`$ git reset --hard HEAD^`		  <!--回退至上一个版本-->

​		`$ git reset --hard HEAD^`		  <!--回退至上上个版本-->

​		`$ git reset --hard HEAD~?`		<!--回退至某一个版本,?代表往上?个版本-->

​		`$ git reset  --hard commit_ID`     	<!--回退至对应commit_ID的版本-->

​		`$ git reflog`					  <!--查看命令历史，确定要回退版本的commit_ID-->

​		`$ git log` 					     <!--显示提交日志,详细--> 

​		`$ git log --pretty=oneline`		    <!--日志,简化版(commit id和修改内容)-->

##### 2.工作区VS暂存区

​	`git add` 将工作区修改和新加的文件添加到暂存区

​	`git commit` 将暂存区文件一次性提交到分支

##### 3.管理修改

​	git跟踪文件修改，每次修改若不用git add 添加到暂存区，就不会加入到commit中。

​	`$ cat fie_name` 				<!--查看文件内容-->

​	修改文件内容

​	`$ git add file_name` 			 <!--添加-->

​	修改文件内容

​	`$ git commit -m file_nam`e		<!--提交暂存区文件-->

​	`$ git status` 					<!--查看当前git状态，modified文件有修改未提交-->

​	``$ git diff HEAD -- file_name`` 命令可以查看工作区和版本库里面最新版本的区别

##### 4.撤销修改

​	场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令`git checkout -- file`。

​	场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，	第一步用命令`git reset HEAD <file>`，就回到了场景1，第二步按场景1操作。

​	场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，前提是没有推送到远程库。使用

​	`$ git reset --hard commit_ID($ git reset --hard HEAD^/HEAD^^/HEAD~?`--？为数字)

##### 5.删除文件

​	场景一：已提交版本库，误删本地文件，需要恢复

​		`$ git  checkout -- file_name` 时其实是将版本库的版本替换工作区的版本，所以，无论工作区是修改还是删除都		可以还原

​	场景二：工作区要删除文件已提交至版本库（本地非远程）

​		`$ git rm file_name` 

​		`$ git commit` 		

​	注意：从本地版本库恢复的文件会丢失最近一次提交后修改的内容。

### 三	远程仓库

##### 1.推送到远程仓库

​	`$ ssh-keygen -t rsa -C "youremail@example.com"`     生成公钥,在github添加

​	`$ git remote add origin git@github.com:DumboJ/LxfGit.git`

​	关联一个远程库，使用命令`git remote add origin git@server-name:path/repo-name.git`；

​	关联后，使用命令`git push -u origin master`第一次推送master分支的所有内容(`-u`参数，Git不但会把本地的	`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令)

​	注:首次提交时版本库并没有远程仓库中的README.md文件,	在push前先执行	`git pull --rebase origin master`

​	此后，每次本地提交后，只要有必要，就可以使用命令`git push origin master`推送最新修改；

2. ##### 从远程库克隆

   要克隆一个仓库，确认仓库地址，然后使用`git clone`命令克隆。

   ```
   不指定分支：
   git + clone + clone_address
   例：git clone git@github.com:Dumboj/DmboJ.github.io.git
   
   指定分支：
   git + clone + -b + branch_name + clone_address
   例：git clone -b hexo git@github.com:Dumboj/DmboJ.github.io.git
   ```

   Git支持多种协议，包括`https`，但通过`ssh`支持的原生`git`协议速度最快。

### 四  分支管理

每次提交,git 都串成一条时间线.git 的分支指向提交,HEAD指向当前分支.

Git鼓励大量使用分支：查看分支：`git branch`		创建分支：`git branch <name>`	

​		切换分支：`git checkout <name>`		创建+切换分支：`git checkout -b <name>`

合并某分支到当前分支：`git merge <name>`		删除分支：`git branch -d <name>`

##### 1.创建与合并分支

###### ①:创建分支

​		`A-$ git branch branch_name`	<!--创建分支-->

​		`B- $ git checkout branch_name`	<!--切换分支-->

​		相当于 `$  git checkout -b branch_name`		<!--参数-b表示创建并切换分支-->

​	`$ git branch` 		<!--查看当前分支,列出所有分支,当前分支前有*-->

###### ②合并分支

​		`git merge branch_name`	<!--合并指定分支到当前分支-->

###### ③删除分支

​	`$ git branch -d branch_name`

##### 2.解决冲突

​	当前分支和合并分支操作相同文件都提交,合并时会造成冲突

​	当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。

​	解决冲突就是把Git合并失败的文件手动编辑为我们希望的内容，再提交。

​	用`git log --graph`命令可以看到分支合并图。

##### 3.分支管理策略

​	合并分支时，如果可能，Git会用`Fast forward`模式，但这种模式下，删除分支后，会丢掉分支信息。



​	合并分支时,不用`Fast forward`模式,则	`$ git merge --no-ff -m "merge with no-ff" branch_name`



​	合并分支时，加上`--no-ff`参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而`fast forward`合并就看不出来曾经做过合并。

##### 4.bug分支

​	修复bug时，我们会通过创建新的bug分支进行修复，然后合并，最后删除；

​	先把工作现场`git stash`一下，然后去修复bug，修复后，再`git stash pop`，回到工作现场。

​	`$ git stash list` 可以查看stash内容.

​	工作现场恢复方式

​		`A.$ git stash apply` 恢复,恢复后,stash 内容并不删除,需要用git stash drop来删除.

​		`B.$ git stash pop` 恢复的同时把stash内容也删了.

##### 5.Feature 分支

​	开发一个新feature，最好新建一个分支；

​	如果要丢弃一个没有被合并过的分支，可以通过`git branch -D <name>`强行删除。

##### 6.多人协作

​	推送分支,实际就是将分支上的所有本地提交推送到远程库.推送时,指定本地分支,这样,git 就会把该分支推送到远程库对应的远程分支上:

​	`$ git push origin branch_name`		<!--origin远程仓库的默认名称-->

​	`$ git clone git@github.com:user_name/respository_name.git`

1. 首先，可以试图用`git push origin <branch-name>`推送自己的修改；

2. 如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并；

3. 如果合并有冲突，则解决冲突，并在本地提交；

4. 没有冲突或者解决掉冲突后，再用`git push origin <branch-name>`推送就能成功！

   如果`git pull`提示`no tracking information`，则说明本地分支和远程分支的链接关系没有创建，用命令`git branch --set-upstream-to <branch-name> origin/<branch-name>`

- 查看远程库信息，使用`git remote -v`；
- 本地新建的分支如果不推送到远程，对其他人就是不可见的；
- 从本地推送分支，使用`git push origin branch-name`，如果推送失败，先用`git pull`抓取远程的新提交；
- 在本地创建和远程分支对应的分支，使用`git checkout -b branch-name origin/branch-name`，本地和远程分支的名称最好一致；
- 建立本地分支和远程分支的关联，使用`git branch --set-upstream branch-name origin/branch-name`；
- 从远程抓取分支，使用`git pull`，如果有冲突，要先处理冲突。

##### 7.rebase

- rebase操作可以把本地未push的分叉提交历史整理成直线；
- rebase的目的是使得我们在查看历史提交的变化时更容易，因为分叉的提交需要三方对比。

### 五.标签管理

##### 1.创建标签

​		命令`git tag <tagname>`用于新建一个标签，默认为`HEAD`，也可以指定一个commit id；

​		命令`git tag -a <tagname> -m "blablabla..."`可以指定标签信息；

​		命令`git tag`可以查看所有标签。

##### 2.操作标签

​	命令`git push origin <tagname>`可以推送一个本地标签；

​	命令`git push origin --tags`可以推送全部未推送过的本地标签；

​	命令`git tag -d <tagname>`可以删除一个本地标签；

​	命令`git push origin :refs/tags/<tagname>`可以删除一个远程标签。

# SSH 传输设置

Git 仓库和 Github 中心仓库之间的传输是通过 SSH 加密。

如果工作区下没有 .ssh 目录，或者该目录下没有 id_rsa 和 id_rsa.pub 这两个文件，可以通过以下命令来创建 SSH Key：

```
$ ssh-keygen -t rsa -C "youremail@example.com"
```

然后把公钥 id_rsa.pub 的内容复制到 Github "Account settings" 的 SSH Keys 中。





# .gitignore 文件

忽略以下文件：

- 操作系统自动生成的文件，比如缩略图；
- 编译生成的中间文件，比如 Java 编译产生的 .class 文件；
- 自己的敏感信息，比如存放口令的配置文件。

不需要全部自己编写，可以到 [https://github.com/github/gitignore](https://github.com/github/gitignore) 中进行查询。

# Git 命令一览

<div align="center"> <img src="https://gitee.com/CyC2018/CS-Notes/raw/master/docs/pics/7a29acce-f243-4914-9f00-f2988c528412.jpg"/> </div><br>

比较详细的地址：http://www.cheat-sheets.org/saved-copy/git-cheat-sheet.pdf

