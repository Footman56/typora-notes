brew是Mac、linux上快速下载，运行的工具，**就不需要再将下载好的软件移动到项目下**

![image-20230725225214883](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202307252252208.png)

# 零、安装brew

```sh
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

- 尽量在bash或者zsh下安装，fish下会提示不识别'$'

- 不需要使用超级权限（sudo），该文件会将HomeBrew安装至`usr/local`目录下。安装过程中会提示你执行哪些动作

  安装完成后，通过

  ```sh
  brew -v 
  ```

  查看安装的版本

  | formula (e) | 安装包的描述文件，formulae 为复数                            |
  | ----------- | ------------------------------------------------------------ |
  | cellar      | 安装好后所在的目录                                           |
  | keg         | 具体某个包所在的目录，keg 是 cellar 的子目录                 |
  | bottle      | 预先编译好的包，不需要现场下载编译源码，速度会快很多；官方库中的包大多都是通过 bottle 方式安装 |
  | tap         | 下载源，可以类比于 Linux 下的包管理器 repository             |
  | cask        | 安装 macOS native 应用的扩展，你也可以理解为有图形化界面的应用。 |
  | bundle      | 描述 Homebrew 依赖的扩展                                     |

# 一、常用指令

```sh
# 下载软件
brew install [servie_name]
# 删除软件
brew uninstall [servie_name]
# 查看下载列表
brew list 
# 搜索软件
brew search 
# 查看某个特定软件的信息
brew info [servie_name] 
```

```sh
# 更新 brew
brew update

# 升级所有已过时的软件，即列出的已过时软件
brew upgrade
# 升级指定的软件
brew upgrade [servie_name] 
```

```sh
# 清理指定的软件过时包
brew cleanup [servie_name]  
# 清理所有的过时软件
brew cleanup 
```

# 二、使用国内源

```sh
# 清华源 
# 替换brew.git
cd "$(brew --repo)"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
# 替换homebrew-core.git
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
# 刷新源
brew update
```

```sh
# 阿里源
# 替换brew.git
cd "$(brew --repo)"
git remote set-url origin https://mirrors.aliyun.com/homebrew/brew.git
# 替换homebrew-core.git
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.aliyun.com/homebrew/homebrew-core.git
# 刷新源
brew update
```

# 三、搜索

```sh
# 帮助我们搜索一些应用
brew search [keyword]
```

也可以在https://formulae.brew.sh/ 中输入关键字，之后查看安装

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220913232328158.png" alt="image-20220913232328158" style="zoom:50%;" />

# 四、管理后台软件

一些软件是有服务器和客户端区别的，我们安装好服务器后，就需要启动，才能保证客户端正常使用。

后台启动软件（nginx、mysql）

```sh
# 查看所有服务
brew services list
# 单次运行某个服务
brew services run  [servie_name] 
# 运行某个服务，并设置开机自动运行
brew services start  [servie_name] 
# 停止某个服务
brew services stop  [servie_name] 
# 重启某个服务
brew services restart [servie_name] 
```



# 五、检查brew

应用自检

```sh
brew doctor 
```

# 六、辅助下载

一些常用的top 能够帮助我们下载

## caskroom

brew 会默认安装，直接使用就可以

能够下载一些图形化软件

```
brew  install  --cask [软件名] (bigsur)
```

可以在https://formulae.brew.sh/cask/ 查看支持的cask 或者从官网正规渠道下载https://macappstore.org/

### Cakebrew

Cakebrew 是 Homebrew 的 GUI 管理器，在 Cakebrew 中，你可以看到当前所有已经安装的软件，并可以在 Caskbrew 中对其他软件执行升级等操作。

你只需要执行 `brew cask install cakebrew` 就可以完成 Cakebrew 的安装。

安装完成后，在 LaunchPad 中打开即可。

### launchrocket

launchrocket 可以用于管理 Homebrew 安装的服务，在使用时，你需要先添加对应的tap，然后再安装软件。

```
brew tap jimbojsb/launchrocket
brew install --cask launchrocket
```

**已经不维护啦，在monterey 中无法使用**

# 七、缓存

```sh
# 查看brew 缓存位置
brew --cache
# 进入缓存目录
cd $(brew --cache)
```

# 八、卸载

```sh
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)"
```

如果比较耗时的话可以采用

```sh
/bin/bash -c "$(curl -fsSL https://gitee.com/ineo6/homebrew-install/raw/master/uninstall.sh)"
```

