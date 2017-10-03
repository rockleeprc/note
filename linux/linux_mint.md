
## ssh

ssh 172.16.1.11 -l root


## 更新命令


	sudo apt-get update

	# 将系统升级到新版本
	apt-get dist-upgrade
	# 更新软件包
	sudo apt-get upgrade

	# 安装一个新软件包（参见下文的aptitude）
	apt-get install packagename

	# 卸载一个已安装的软件包（保留配置文档）
	apt-get remove packagename

	# 卸载一个已安装的软件包（删除配置文档）
	apt-get remove --purge packagename

	# 删除包及其依赖的软件包
	apt-get autoremove packagename

	# 删除包及其依赖的软件包+配置文件，比上面的要删除的彻底一点
	apt-get autoremove --purge packagname

	# 有些软件很难卸载，而且还阻止了别的软件的应用，就能够用这个，但是有点冒险
	dpkg --force-all --purge packagename

	# 在软件包列表中搜索字符串
	apt-cache search string

	# 显示软件包信息
	apt-cache showpkg pkgs

	# 打印可用软件包列表
	apt-cache dumpavail

## 修改所有者和组

	chown -R 用户名 目录名
	chgrp -R 组名 目录名


## gcc 安装

	gcc --version

	apt-get install gcc
	apt-get install g++
	apt-get install make
	apt-get install cmake

	gcc *.c -c *.out
	./*.out

## mysql
	sudo apt-get install mysql-server

## virtualbox
	apt-get install virtualbox
	apt-get install virtualbox-dkms
	apt-get install virtualbox-source
	apt-get install virtualbox-qt

	apt-get install virtualbox
	apt-get install virtualbox-dkms virtualbox-source virtualbox-qt
