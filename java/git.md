## 什么是git

cvs/svn：集中式的版本控制系统，版本库存放在中央服务器

git：分布式版本控制系统，版本库存放在本地

## git安装

	# Debian安装
	sudo apt-get install git
	
	# 配置用户名和邮箱 --global参数表示本台机器上的所有git仓库都使用这个配置
	git config --global user.name "Your Name"
	git config --global user.email "email@example.com"

### 新环境配置
	# 设置用户名
	$ git config --global user.name "yan.li"
	# 设置邮箱
	$ git config --global user.email "yan.li@tadu.com"
	# windowns 生成rsa
	# ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
	# https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/
	$ ssh-keygen.exe -C "yan.li@tadu.com" -t rsa
	Generating public/private rsa key pair.
	Enter file in which to save the key (/c/Users/Administrator/.ssh/id_rsa):
	Enter passphrase (empty for no passphrase):
	Enter same passphrase again:
	Your identification has been saved in /c/Users/Administrator/.ssh/id_rsa.
	Your public key has been saved in /c/Users/Administrator/.ssh/id_rsa.pub.
	The key fingerprint is:
	SHA256:thfH2u+0ilLnNhXemwWdf2LzDEsWB1K+Ne8ZO0KCSvA yan.li@tadu.com
	The key's randomart image is:
	+---[RSA 2048]----+
	|             ..  |
	|            ...  |
	|     .       ..+o|
	|      o   ..  ++=|
	|       ES...oo.Bo|
	|      .....=+ X.O|
	|        .. .+oo++|
	|         ....++  |
	+----[SHA256]-----+

## 查看配置

```
# 查看系统config
git config --system --list
# 查看当前用户（global）配置
git config --global  --list
# 查看当前仓库配置信息
git config --local  --list
```



## 用户名修改

```
$  git config --global user.name "输入你的用户名"
$  git config --global user.email "输入你的邮箱"

$  git config --global --replace-all user.email "输入你的邮箱" 
$  git config --global --replace-all user.name "输入你的用户名"

```



## 配置全英文

	# .bashrc中加入 git log时有中文乱码问题
	alias git='LANG=en_GB git'
	# git全英文信息提示 git log时中文没有乱码
	alias git='LANG=en_US.UTF-8 git'

## 创建版本库

	# 仓库目录
	$ mkdir learngit
	$ cd learngit/
	
	# 初始化目录为git可以管理的仓库 初始化后learngit目录下会多出一个.git目录 用来管理跟踪版本库的
	$ git init

## 文件添加到版本库

	# 工作区文件提交到Stage
	$ git add readme.txt
	
	# Stage文件提交到版本库 -m表示提交说明
	$ git commit -m "wrote a readme file"
	[master （根提交） 38cfbae] wrote a readme file
	1 file changed, 1 insertion(+)
	create mode 100644 readme.txt
	
	# git提交分为两部 1.add 2.commit
	# 可以add多个文件 一次commit提交
	git add f1.txt f2.txt
	
	$ git commit -m "wrote 2 files"
	[master 1576f69] wrote 2 files
	2 files changed, 3 insertions(+)
	create mode 100644 f1.txt
	create mode 100644 f2.txt

提交过程：工作区 --> Stage --> master分支

## 查看版本库状态

	# 查看版本库状态
	 $ git status
	位于分支 master
	尚未暂存以备提交的变更：
	  （使用 "git add <文件>..." 更新要提交的内容）
	  （使用 "git checkout -- <文件>..." 丢弃工作区的改动）
	
		修改：     readme.txt
	
	修改尚未加入提交（使用 "git add" 和/或 "git commit -a"）
	
	# 查看workspace和stage的差异 +代表新增的内容
	$ git diff
	diff --git a/readme.txt b/readme.txt
	index b2a57ff..a47104d 100644
	--- a/readme.txt
	+++ b/readme.txt
	@@ -1 +1,4 @@
	-This is a readme.txt file for git.
	+This is a test readme.txt file for git.
	+
	+update
	+create
	
	# 查看stage和版本库的差异（分支）
	$ git diff head 

## 日志查看

	# 查看commit日志
	$ git log
	commit 0b858d00cc1935b46ccad7fba76a058af8a19aa8
	Author: rockleeprc <rockleeprc@outlook.com>
	Date:   Sat Jun 3 12:15:58 2017 +0800
	
	    update readme.txt
	
	commit 1576f693898a8719d8460bd4064e787f9644dda5
	Author: rockleeprc <rockleeprc@outlook.com>
	Date:   Sat Jun 3 11:46:54 2017 +0800
	
	    wrote 2 files
	
	commit 38cfbae1444108849ae1e7f9eebfa9136a645c99
	Author: rockleeprc <rockleeprc@outlook.com>
	Date:   Sat Jun 3 11:41:22 2017 +0800
	
	    wrote a readme file
	
	# 查看该文件相关的commit记录
	$ git log jcip.md
	
	# 显示该文件每次提交的diff
	$ git log -p jcip.md
	
	# 查看某次提交中的某个文件变化
	$ git show dd8f347c6689dc24a6e9ebae24655a794a07f3bb jcip.md
	
	# 根据commit-id查看某个提交
	$ git show dd8f347c6689dc24a6e9ebae24655a794a07f3bb
	
	# 每次commit日志信息在一行输出
	$ git log --pretty=oneline
	0b858d00cc1935b46ccad7fba76a058af8a19aa8 update readme.txt
	1576f693898a8719d8460bd4064e787f9644dda5 wrote 2 files
	38cfbae1444108849ae1e7f9eebfa9136a645c99 wrote a readme file
	
	# 查看文件文件历史记录及修改
	$ git log --pretty=oneline tadu-ios-server2.iml
	$ git show af8b96e6b9723009701286d4e29f7568ff0b300a tadu-ios-server2.iml


## 版本回退

	# 当前版本是HEAD 上一个版本是HEAD^
	$ git reset --hard HEAD^
	HEAD 现在位于 0b858d0 update readme.txt
	
	# 根据id回退
	$ git reset --hard 38cfbae1444108849ae1e7f9eebfa9136a645c99
	# 强行push
	$ git push --force
	
	# 查看每一次命令记录
	$ git reflog
	38cfbae HEAD@{0}: reset: moving to 38cfbae1444108849ae1e7f9eebfa9136a645c99
	0b858d0 HEAD@{1}: reset: moving to HEAD^
	b90457c HEAD@{2}: commit: wrote callback
	0b858d0 HEAD@{3}: commit: update readme.txt
	1576f69 HEAD@{4}: commit: wrote 2 files
	38cfbae HEAD@{5}: commit (initial): wrote a readme file
	
	# 回退到指定版本
	git reset --hard b90457c
	HEAD 现在位于 b90457c wrote callback
	
	# 回退代码
	git reset --hard the_commit_id //把the_branch本地回滚到the_commit_id
	git push -f origin develop 强制本地代码覆盖线上代码

## 撤销修改

	# readme.txt自修改后还没有add到stage，现在，撤销修改就回到和版本库一模一样的状态
	# readme.txt已经添加到stage后，又作了修改，现在，撤销修改就回到添加到stage后的状态
	$ git checkout -- readme.txt
	
	# 撤销Stage中的修改 把Stage的修改重新放回工作区 然后使用git checkout --彻底还原
	$ git reset HEAD readme.txt
	重置后取消暂存的变更：
	
	# 查看工作区和版本库里面最新版本的区别
	$ git diff HEAD -- readme.txt
	diff --git a/readme.txt b/readme.txt
	index 448436e..6c283cd 100644
	--- a/readme.txt
	+++ b/readme.txt
	@@ -2,3 +2,4 @@ This is a test readme.txt file for git.
	 call back
	 update
	 create
	+delete

* 撤销工作区修改：
	* git checkout -- file

* 撤销Stage修改:
	* git reset HEAD file
	* git checkout -- file

## 删除

	# 删除test.txt文件 如果需要恢复使用git checkout -- test.txt
	$ git status
	位于分支 master
	尚未暂存以备提交的变更：
	  （使用 "git add/rm <文件>..." 更新要提交的内容）
	  （使用 "git checkout -- <文件>..." 丢弃工作区的改动）
	
		删除：     test.txt
	
	修改尚未加入提交（使用 "git add" 和/或 "git commit -a"）
	
	# 从版本库中删除
	$ git rm test.txt
	rm 'test.txt'
	# 从git中删除，文件系统中保留
	$ git reset filename
	
	# 提交到分支 彻底删除
	$ git commit -m "rm test.txt"
	[master 5c6c45c] rm test.txt
	 1 file changed, 1 deletion(-)
	 delete mode 100644 test.txt


## 提交分支到远程仓库

	# 本地仓库关联远程仓库
	$ git remote add origin git@github.com:rockleeprc/learngit.git
	
	# -u把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来
	$ git push -u origin master
	# 推送最新的修改
	$ git push origin master
	# 提交本地分支到远程仓库
	$ git push origin local_branch:remote_branch
	$ git push origin feature/original_publishing:feature/original_publishing
	$ git push origin featurn/slowquery

## .gitignore配置

由于gitignore只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的

	# 有时候需要突然修改 .gitignore 文件，随后要立即生效
	git rm -r --cached .  #清除缓存  
	git add . #重新trace file  
	git commit -m "update .gitignore" #提交和注释  
	git push origin master #可选，如果需要同步到remote上的话  


## 创建分支

	# 创建分支
	$ git branch dev
	# 切换分支
	$ git checkout dev
	Switched to branch 'dev'
	
	# 以上命令可以合并为一条命令 -b创建dev分支后切换到dev上
	$ git checkout -b dev
	切换到一个新分支 'dev'
	
	# 切换到远程的一个分支
	$ git checkout -b feature/ios2.40_mitao origin/feature/ios2.40_mitao


## 分支查看

	# 查看分支
	$ git branch
	* dev
	
	# 查看当前分支从哪个分支
	$ git branch -vv
	* develop          4d7ddbf [origin/develop] Merge branch 'develop' into feature/qaccommunity_myCount
	  feature/cms_mail 026eb8d [origin/feature/cms_mail] 邮箱密码过期，修改邮箱密码
	  master           203eee8 [origin/master] init
	
	git log --graph --all --decorate
	git reflog --date=local | grep <branchname>
	
	# 查看远程分支
	$ git branch -a
	* develop
	  feature/cms_mail
	  master
	  remotes/origin/HEAD -> origin/master
	  remotes/origin/develop
	  remotes/origin/feature/add_advert_channel
	  remotes/origin/feature/add_ip_sync_file
	  remotes/origin/feature/android3.80_bookPartError_youhua
		remotes/origin/feature/cms_interactive_fiction
		remotes/origin/feature/cms_ios2.0
		remotes/origin/feature/cms_ios2.20_umeng
		remotes/origin/feature/cms_mail
		remotes/origin/feature/cms_miss_package
		remotes/origin/feature/cms_red_package_youhua
		remotes/origin/feature/cms_test
	
	$ git br -al
	develop
	feature/cms_mail
	* feature/console_log
	feature/edit_name
	feature/feature/bookbar_select_optimize
	featurn/slowquery
	master
	remotes/origin/HEAD -> origin/master
	remotes/origin/develop
	remotes/origin/feature/add_advert_channel
	remotes/origin/feature/add_ip_sync_file


## 分支合并

	# msater分支
	$ git status
	位于分支 master
	您的分支与上游分支 'origin/master' 一致。
	无文件要提交，干净的工作区
	
	# 合并dev分支到当前分支 “Fast-forward”是直接把master指向dev的当前提交
	$ git merge dev
	更新 a5d884d..7e9121b
	Fast-forward
	 gitcommd.txt | 1 +
	 1 file changed, 1 insertion(+)
	
	 # 别人提交到自己建的featurn/slowquery，进行拉取
	 $ git pull origin featurn/slowquery featurn/slowquery


	# 分支合并图			 
	$ git log --graph
	
	# 合并feature/feature/bookbar_select_optimize分支到feature/console_log
	# 1.checkout一份要合并的远程分支
	$ git checkout feature/feature/bookbar_select_optimize
	# 2.切换到原分支
	# $ git checkout feature/console_log
	# 3.在feature/console_log分支上，把feature/feature/bookbar_select_optimize合并过来
	$ git merge feature/feature/bookbar_select_optimize
	# 4.push到仓库
	$ git push origin feature/console_log

## 删除分支

	# 删除本地已经合并的分支
	$ git branch -d dev
	已删除分支 dev（曾为 7e9121b）。
	
	# 删除本地未合并的分支
	$ git branch -D dev
	
	# 删除远程分支
	$ git push origin --delete dev
	To https://github.com/rockleeprc/firstproject.git
	 - [deleted]         dev

## 冲突解决

	# 创建dev分支 修改gitcommd.txt后commit 在master分支修改gitcommd 在master分支合并dev分支
	$ git merge dev
	自动合并 gitcommd.txt
	冲突（内容）：合并冲突于 gitcommd.txt
	自动合并失败，修正冲突然后提交修正的结果。
	
	# git不能自动合并分支 解决冲突后再提交
	first
	second
	<<<<<<< HEAD #当前分支内容（master）
	brach dev hello nimei
	======= #dev分支内容
	brach dev hello tom
	>>>>>>> dev

## 分支管理策略

	# 分支合并时使用Fast forward模式 删除分支后会丢失分支信息
	 $ git log --graph --pretty=oneline
	*   30f1d4b4739b49288a90367ad0fdfe8004f876ea merge
	|\  
	| * 9eb3452e6fb37f2e839735da5c9464514f6e2d95 hello tom
	* | 3a8904c7a831767f3a38a21917b4b5de47fbc508 hello nimei
	|/  
	* 7e9121bab1a13eb53c1eb9accdd4d66d4716a897 branch test
	* a5d884d9d4fc52ad8bf750d5041c2b51d32ee5a4 add gitcommd.txt
	
	# 禁用Fast forward 合并时要创建一个commit
	$ git merge --no-ff -m "merge no-ff" dev
	Merge made by the 'recursive' strategy.
	 gitcommd.txt | 3 +++
	 1 file changed, 3 insertions(+)
	
	# 禁用Fast forward 查看日志可以看到merge后的分支信息
	$ git log --graph --pretty=onelin
	*   b5e5e04c6627528076b7b1e2b63545331003ece5 merge no-ff
	|\  
	| * 76e91dd8a09244b9741263574dafa23c883da34b branch dev
	| * 56224c92ba3ce5a345f2d54bafdb9ed1b0adb636 branch dev
	|/  
	*   30f1d4b4739b49288a90367ad0fdfe8004f876ea merge
	|\  
	| * 9eb3452e6fb37f2e839735da5c9464514f6e2d95 hello tom
	* | 3a8904c7a831767f3a38a21917b4b5de47fbc508 hello nimei
	|/  
	* 7e9121bab1a13eb53c1eb9accdd4d66d4716a897 branch test
	* a5d884d9d4fc52ad8bf750d5041c2b51d32ee5a4 add gitcommd.txt


## stash

	# 隐藏工作区未提交的内容到stash
	$ git stash
	Saved working directory and index state WIP on master: b5e5e04 merge no-ff
	HEAD is now at b5e5e04 merge no-ff
	
	# 查看
	$ git stash list
	stash@{0}: WIP on master: b5e5e04 merge no-ff
	
	# 把stash内容恢复到工作去 删除stash内容
	$ git stash pop
	On branch master
	Your branch is ahead of 'origin/master' by 6 commits.
	  (use "git push" to publish your local commits)
	Changes not staged for commit:
	  (use "git add <file>..." to update what will be committed)
	  (use "git checkout -- <file>..." to discard changes in working directory)
	
		modified:   gitcommd.txt
	
	no changes added to commit (use "git add" and/or "git commit -a")
	Dropped refs/stash@{0} (20b6b120142c4dbeb26f7ea2704380e5447bae79)
	
	# stash内容恢复到工作区 但不删除stash里的内容
	git stash apply
	# 删除stash内容
	git stash drop


## 远程仓库

	# 查看远程仓库信息 远程仓库默认是origin
	$ git remote
	origin
	# -v参数查看更详细的信息
	$ git remote -v
	
	# 推送到远程仓库的master分支
	$ git push origin master
	
	# 抓取远程仓库
	$ git pull
	
	# 建立本地分支和远程分支的关系
	$ git branch --set-upstream branch-name origin/branch-name
	
	$ git fetch origin master # 将远程仓库的master分支下载到本地当前branch中
	$ git diff master origin/master # 当前master分支和仓库master比较
	$ git log -p master  ..origin/maste	# 比较本地的master分支和origin/master分支的差别
	$ git merge origin/master # 合并master到当前分支
	$ git pull origin master # 从远程获取最新版本并merge到本地

# todo标签管理

# 分支命名规范

- **master** ：主分支，暂不使用
- **deployment** ：线上发布的稳定版本分支，上线后的版本要合并到该分支
- **feature**/version/${版本号} ：版本迭代分支
  - feature/version/1.0.0 ：1.0.0 版本开发
  - feature/version/1.1.0  ：1.1.0版本开发
- **feature**/${模块/功能} ：模块/功能开发分支，非正式的版本迭代
- **optimize**/${模块/功能名称}_${优化内容} ：优化分支
  - optimize/order_sql_slow_query  ：订单慢查询优化
- **hotfix**/${模块/功能名称}_${bug名称} ：线上紧急bug分支，修复后马上上线的版本
  - hotfix/order_service_npe ：订单模块 service 层空指针异常，`service`不是非必须的
- **fix**/${模块/功能名称}_${bug名称} ：非紧急bug分支，跟随下一个版本发布上线
  - bug/pay_controller_npe ：pay功能 controller 层空指针异常,` controller`不是非必须的

