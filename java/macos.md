
## 安装Homebrew

	# 安装xcode
	xcode-select --install

	# 查看xcode是否安装成功
	$ xcode-select -p
	/Library/Developer/CommandLineTools

	# 安装homebrew
	/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

	mkdir homebrew && curl -L https://github.com/Homebrew/brew/tarball/master | tar xz --strip 1 -C homebrew
	
	# 添加环境变量
	$ echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.bash_profile

	# Cmd+T 打开一个新的 terminal
	$ brew doctor  

* 安装完成后，Homebrew 会将本地 /usr/local 初始化为 git 的工作树，并将目录所有者变更为当前所操作的用户，将来 brew 的相关操作不需要 sudo 


## HomeBrew基本使用
Homebrew Cask，它是一套建立在 Homebrew 基础之上的 OS X 软件安装命令行工具，是 Homebrew 的扩展



	# 安装软件：brew install 软件名，例：
	brew install wget
	
	# 搜索软件：brew search 软件名，例：
	brew search wget
	
	# 卸载软件：brew uninstall 软件名，例：
	brew uninstall wget

	# 查看你的包是否需要更新：
	$ brew outdated  
	
	# 更新具体软件：brew upgrade 软件名 ，例：
	brew upgrade git

	# 更新所有软件：
	brew update

	# Homebrew 将会把老版本的包缓存下来，以便当你想回滚至旧版本时使用。但这是比较少使用的情况，当你想清理旧版本的包缓存时，可以运行：
	$ brew cleanup  

	# 显示已安装软件：
	brew list
	# 查看你安装过的包列表（包括版本号）：
	$ brew list --versions  

	# 查看软件信息：brew info／home 软件名 ，例：
	brew info git ／ brew home git

	# 显示包依赖：
	brew deps
	brew reps

	# 显示安装的服务：
	brew services list
	# 安装服务启动、停止、重启：
	brew services start/stop/restart serverName

## 安装Homebrew Cask

	# 添加 Github 上的 caskroom/cask 库
	$ brew tap caskroom/cask
	
	# 安装 brew-cask  
	$ brew install brew-cask  

	# 安装 Google 浏览器
	$ brew cask install google-chrome 

	# 查看软件相关信息:brew cask info 软件名
	brew cask info google-chrome

	# 卸载：
	brew cask uninstall + 文件名

	# 删除 Homebrew Cask 下载的包
	brew cask cleanup
	
	# 列出通过 Homebrew Cask 安装的包
	brew cask list

	# 更新
	$ brew update && brew upgrade brew-cask && brew cleanup
 
	# 如果你想查看 cask 上是否存在你需要的 app，可以到 caskroom.io 进行搜索。

	# 文件预览插件：有些 插件 可以让 Mac 上的文件预览更有效，比如语法高亮、markdown 渲染、json 预览等等
	$ brew cask install qlcolorcode
	$ brew cask install qlstephen
	$ brew cask install qlmarkdown
	$ brew cask install quicklook-json
	$ brew cask install qlprettypatch
	$ brew cask install quicklook-csv
	$ brew cask install betterzipql
	$ brew cask install webp-quicklook
	$ brew cask install suspicious-package   