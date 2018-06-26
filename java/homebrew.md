

*安装xcode*

```bash
xcode-select --install
```

*查看xcode是否安装成功*

```bash
$ xcode-select -p
/Library/Developer/CommandLineTools
```

*安装homebrew*

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

*查看homebrew健康*

```bash
$ brew doctor  	
```

安装完成后，Homebrew 会将本地 /usr/local 初始化为 git 的工作树，并将目录所有者变更为当前所操作的用户，将来 brew 的相关操作不需要 sudo 

Homebrew安装成功后，会自动创建目录 **/usr/local/Cellar**来存放Homebrew安装的程序

 *安装软件：brew install 软件名*

```
brew install wget
```

 *搜索软件：brew search 软件名*
```bash
brew search wget
```

 *卸载软件：brew uninstall 软件名*
```bash
brew uninstall wget
```
 *显示已安装软件*
```bash
brew list
```
 *查看安装过的包列表（包括版本号）*
```bash
$ brew list --versions  
```

 *查看软件信息*

```
brew info git
```

 *显示安装的服务*

```bash
brew services list
```
 *安装服务启动、停止、重启*
```bash
brew services start/stop/restart serverName
```

*检查可用更新*

```bash
brew outdated
brew cask outdated
```

*更新*

```bash
brew upgrade             # 更新所有的包
brew upgrade $FORMULA    # 更新指定的包	
brew cask upgrade
brew cask upgrade $FORMULA
```

*清理就版本*

```bash
brew cleanup             # 清理所有包的旧版本
brew cleanup $FORMULA    # 清理指定包的旧版本
brew cleanup -n          # 查看可清理的旧版本包，不执行实际操作
```

*锁定不更新的包*

```bash
brew pin $FORMULA      # 锁定某个包
brew unpin $FORMULA    # 取消锁定
```

 *更新所有软件*

```
brew update
```

*查看已安装的包的依赖，树形显示*

```bash
brew deps --installed --tree 	
```
*显示包依赖*

```bash
brew deps
brew reps
```

