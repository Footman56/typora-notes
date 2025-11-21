下载完iterm2 之后样式不行，需要使用zsh 增强

一、下载zsh

```
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202311231634670.png" alt="image-20231123163400991" style="zoom:50%;" />



二、自选主题、字体、配色



三、指令高亮

```
cd ~/.oh-my-zsh/custom/plugins/
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git
vi ~/.zshrc
```

```.zshrc
plugins=(git
zsh-syntax-highlighting
)
zsh-syntax-highlighting 一定要放在最后
```

```
source ~/.zshrc
```

四、指令自动补全

```bash
cd ~/.oh-my-zsh/custom/plugins/
git clone https://github.com/zsh-users/zsh-autosuggestions.git
vi ~/.zshrc
```

```
plugins=(git
zsh-autosuggestions
zsh-syntax-highlighting
)
zsh-syntax-highlighting 一定要放在最后
```

```
source ~/.zshrc
```

五、默认使用zsh 作为终端

```
chsh -s /bin/zsh
```

```
/Users/peilizhi/.bash_profile
```



六、所有指令失效

1. 在终端输入

   ```sh
   PATH=/bin:/usr/bin:/usr/local/bin:${PATH}
   
   export PATH
   ```

   可以强制恢复其他指令的使用。

2. 打开.zshrc文件

   ```sh
   PATH=/bin:/usr/bin:/usr/local/bin:${PATH}
   export PATH
   
   
   source ~/.bash_profile
   ```

   这一步就在zshrc执行时将 .bash_profile 全部环境变量加入了zsh shell,如果原先没有创建过.bash_profile这个文件就不用加上了,输入步骤二的两行就行,保存后退出

3. 在终端输入

   ```sh
   source .zshrc
   ```