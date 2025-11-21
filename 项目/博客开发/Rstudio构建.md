# 一、安装依赖包

在RStudio 中安装`blogdown` 包

输入命令：

```R
install.packages("blogdown")
```

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221122215323254.png" alt="image-20221122215323254" style="zoom:50%;" />

安装hugo 

```shell
brew install hugo 
```

下载会很慢

# 二、创建Project



FIle -> new Procject ->new Directory -> Website using blogdown

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221122215703230.png" alt="image-20221122215703230" style="zoom:50%;" />

之后成功打开session

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221122223743298.png" alt="image-20221122223743298" style="zoom:50%;" />

# 三、编译

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221122223936442.png" alt="image-20221122223936442" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221122224041552.png" alt="image-20221122224041552" style="zoom:50%;" />

之后就是开始下载一些相关包

至此就可以本地访问了：http://127.0.0.1:4321/ 之后就根据自己的需要做相应的修改

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221123015331314.png" alt="image-20221123015331314" style="zoom:50%;" />

# 四、部署到github

4.1 在github 上新创建一个仓库

4.2 在对应目标创建仓库

```
peilizhideMacBook-Pro:huochai_book peilizhi$ echo "# huochai_book_test" >> README.md
peilizhideMacBook-Pro:huochai_book peilizhi$  git init 
```

4.3 将 本地文件放入暂存区

```
git add .
```

4.4 提交到本地库

```
git commit -m "first commit"

```

4.5 与远程库建立连接 ，**不要建立https 链接**，这个有问题，建立git 链接

```
git branch -M main
git remote add origin git@github.com:Footman56/huochai_book.git
```

4.6 提交到远程库,提交的时候 不能直接提交master分支

```
git push -u origin <branch>
```

4.7 异常时断开与远程服务器连接

```
git remote remove origin
```

# 五、配置netlify

1. 先注册

2. 注册之后选择 sites 

3. 之后没有sites 就需要新建一个

   <img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221123020401694.png" alt="image-20221123020401694" style="zoom:50%;" />



4. 选择github 、选择对应的仓库
5. 设置专属域名

# 六、书写博客推送

选择新建推送

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221123102649589.png" alt="image-20221123102649589" style="zoom:50%;" />

修改些配置信息

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221123102830640.png" alt="image-20221123102830640" style="zoom:50%;" />

创建的时候可以选择目录，更改文件的名称

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221123103111498.png" alt="image-20221123103111498" style="zoom:50%;" />



创建完成之后，通过 `hugo` 命令生成静态网页文件了。

1. 首先切换到项目的主目录

2. 执行 hugo 命令 

   ```
   hugo 
   ```

3. public 目录会生成新的文件

   <img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221125105649260.png" alt="image-20221125105649260" style="zoom:50%;" />

# 遇到的问题

1. 启动的时候找不到 依赖包

   <img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221123012332288.png" alt="image-20221123012332288" style="zoom:50%;" />

   解决方法：

   + ```R
     install.packages("stringi")
     ```

   + 到官网下载，下载之后再拷贝到 依赖库 （/Library/Frameworks/R.framework/Versions/4.2/Resources/library）

     

2. 更换主题

   a. 首先选好想用的主题

   b. 切换到项目目录

   c. 从代码库clone 到当前themes 

   ```
   git clone https://github.com/Y4er/hugo-theme-easybook themes/easybook
   ```

   <img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221123014824158.png" alt="image-20221123014824158" style="zoom:50%;" />

   d. 用主题的config.toml替换原有的config.toml

   ```
   cp themes/easybook/exampleSite/config.toml config.toml
   ```

   

3. 修改主题内的图片文件

   如果页面上提示一个文件404 找不到的话，可以通过source 来查看项目文件位置。发现位置 位于 themes/static 里面，果断将自己的文件也放入  themes/static/images 里面。如果采用hugo 的主题的话，一般资源都要在主题文件里面找，因为他是模版，不能采用实际项目的资源位置

   <img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20221123111720962.png" alt="image-20221123111720962" style="zoom:50%;" />

4. 文件中引入图片的话需要综合文档的位置、图片的位置。

   + 使用 `url` 链接, 添加外部 图片, 这种情况需要一个图床.

   + 使用绝对路径, 添加到站点的 `/static` 目录下，此绝对路径是指 /themes/static
   + 使用相对路径, 添加本地图片, 需要创建与博客markdown文件同名目录, 并将图片放在同名目录中
