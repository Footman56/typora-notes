# 一、准备

安装 Hugo

```shell
brew install hugo 
```

验证hugo 

```sh
hudo version
```

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221125173348521.png" alt="image-20221125173348521" style="zoom:50%;" />

# 二、 创建网站

```
hugo new site huochai-blog
```

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221125173641294.png" alt="image-20221125173641294" style="zoom:50%;" />

# 三、选择主题

在hugo  主题选择适合自己的主题。主题主要就是样式，以及一些附带的功能

```
cd huochai-blog/
git init
git submodule add https://github.com/Footman5
6/theme-stack-huochai.git themes/theme-stack-huochai
```

之后在themes 中就有 hugo-theme-stack

# 四、修改主题文件

每个主题一般都会提供一些实例配置与初始页面，开始使用主题时可以将其 `exampleSite/` 目录下的文件复制到站点目录下，在此基础上进行调整配置。

```sh
cp -rf themes/hugo-theme-stack/exampleSite/* ./
```

之后对config.toml 、config.yaml 根据实际情况修改



# 五、运行

```shell
hugo serve
```

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221125223146151.png" alt="image-20221125223146151" style="zoom:50%;" />

运行之后就可以本地修改，保持就可以直接在 http://localhost:1313/zh-cn/ 查看了

# 六、创建新博客

**要在根目录执行**

```
hugo new posts/my-first-blog.md
```

之后就可以书写博客了

书写的时候是使用markdown 语法的

其中插入图片

！[图片描述]（图片地址）

图片地址：可以选择绝对地址

+ 在根目录创建static/images ,在里面插入图片 

  ![阳光](/images/sun.jpg)

  <img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221125232847728.png" alt="image-20221125232847728" style="zoom:50%;" />

  注意，是只有在执行完 

  ```
  hugo 
  ```

  命令的时候才会讲内容全部放在public 里面，否则的话路径不对
  
  添加图片的时候尽量不要使用<img> 标签，使用原生的i[]()

# 七、创建github 仓库

仓库为  huochai-blog



提交的时候在根目录就可以

```sh
echo "# huochai-blog" >> README.md
git init
# 全部提交
git add .
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:Footman56/huochai-blog.git
git push -u origin main
```



# 八、  Cloudflare Page

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221128115119584.png" alt="image-20221128115119584" style="zoom:50%;" />



选择指定的仓库

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221128115147500.png" alt="image-20221128115147500" style="zoom:50%;" />

选择hugo 方式构建

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221128115318543.png" alt="image-20221128115318543" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221128115335152.png" alt="image-20221128115335152" style="zoom:50%;" />

在这里执行的时候遇到了一直下载原生的stack 主题，但是这个不是我们想要的效果，想要的是使用自定义的主题。所有要把自己定义的主题也放在仓库里面

```sh
git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:Footman56/theme-stack-huochai.git
git push -u origin main
```

再重新选择主题

```sh
git submodule add https://github.com/Footman56/theme-stack-huochai.git themes/theme-stack-huochai
```

同时修改config.yaml 里面的主题

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221128125513714.png" alt="image-20221128125513714" style="zoom:50%;" />

后面修改主题的话，还需要提交的主题仓库里面。（有点麻烦，看看后续没有好的优化方式）



# 九、配置自定义域

![image-20221129023843262](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202211290238298.png)

需要在DNS 中添加一条CNAME配置，这个简单地址执行复杂的项目地址



# 十、自动发布

创建脚本，并授予执行权限

```sh

#!/bin/sh

# If a command fails then the deploy stops
set -e

printf "\033[0;32mDeploying updates to GitHub...\033[0m\n"

# Build the project.
# hugo # if using a theme, replace with `hugo -t <YOURTHEME>`

# Add changes to git.
git add .

# Commit changes.
msg="rebuilding site $(date)"
if [ -n "$*" ]; then
        msg="$*"
fi
git commit -m "$msg"

# Push source and build repos.
git push -f origin main
```

修改后执行

```sh
./deploy.sh
```

每次push 的时候，cloudflare 都会自动部署

