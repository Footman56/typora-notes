需要实现在typora 中实际编写.md ，并且能够支持插入本地图片

# 一、picgo 下载

在官网下载

[picgo]: https://molunerfinn.com/PicGo/

 

# 二、创建github 图床仓库

先创建仓库

token :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

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202301171529242.png" alt="image-20230117152946209" style="zoom:50%;" />



GitHub 上传的时候可能有容量限制，解决方式就是换一个库

# 三、picgo 设置token

仓库名 ： 用户名/仓库名

设定自定义域名：

不填写，加载速度会有点慢，可能会导致你typora的显示很慢甚至显示不出来，但是在博客转存的时候好像没什么问题；

CDN加速：https://cdn.jsdelivr.net/gh/GitHub账户名/仓库名，这个加速国内似乎已经访问不了了，大家可以试试。

fastly加速：https://fastly.jsdelivr.net/gh/GitHub账户名/仓库名，我使用这个在Typora显示图片没问题，但是在转存的时候大部分图片都是转存失败。

gcore加速：https://gcore.jsdelivr.net/gh/GitHub账户名/仓库名 ，这个使用过程中还是有少量的图片转存失败。

![picgo](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202211281800747.png)



选择坚果云来同步数据