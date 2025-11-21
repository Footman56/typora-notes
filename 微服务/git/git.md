

# 一、角色

Gitlab定义了以下几个角色:。
a. Guest - 访客。
b. Reporter - 报告者; 可以理解为测试员、产品经理等，一般负责提交issue等。
c. Developer - 开发者; 负责开发。
d. Master - 主人; 一般是组长，负责对Master分支进行维护。
e. Owner - 拥有者; 一般是项目经理。

# 二、权限

Git Lab中的权限分为访问权限和行为权限两个层次。
**1.访问权限**（Visibility Level）是在建立项目时就需要选定的，主要用于决定哪些人可以访问此项目，包含3种：
a. Public - 公开，任何人可以clone。
b. Private - 私有，只有属于该项目成员才有原先查看。
c. Internal - 内部，拥有Gitlab账号的人都可以clone。
**2.行为权限**：在满足行为权限之前，必须具备访问权限（如果没有访问权限，那就无所谓行为权限了），行为权限是指对该项目进行某些操作，比如提交、创建问题、创建新分支、删除分支、创建标签、删除标签等。

# 三、角色-权限

| 行为                 | Guest | Reporter | Developer | Master | Owner |
| -------------------- | ----- | -------- | --------- | ------ | ----- |
| 创建issue            | ✓     | ✓        | ✓         | ✓      | ✓     |
| 留言评论             | ✓     | ✓        | ✓         | ✓      | ✓     |
| 更新代码             |       | ✓        | ✓         | ✓      | ✓     |
| 下载工程             |       | ✓        | ✓         | ✓      | ✓     |
| 创建代码片段         |       | ✓        | ✓         | ✓      | ✓     |
| 创建合并请求         |       |          | ✓         | ✓      | ✓     |
| 创建新分支           |       |          | ✓         | ✓      | ✓     |
| 提交代码到非保护分支 |       |          | ✓         | ✓      | ✓     |
| 强制提交到非保护分支 |       |          | ✓         | ✓      | ✓     |
| 移除非保护分支       |       |          | ✓         | ✓      | ✓     |
| 添加tag              |       |          | ✓         | ✓      | ✓     |
| 创建wiki             |       |          | ✓         | ✓      | ✓     |
| 管理issue处理者      |       |          | ✓         | ✓      | ✓     |
| 管理labels           |       |          | ✓         | ✓      | ✓     |
| 创建里程碑           |       |          |           | ✓      | ✓     |
| 添加项目成员         |       |          |           | ✓      | ✓     |
| 提交保护分支         |       |          |           | ✓      | ✓     |
| 使能分支保护         |       |          |           | ✓      | ✓     |
| 修改/移除tag         |       |          |           | ✓      | ✓     |
| 编辑工程             |       |          |           | ✓      | ✓     |
| 添加deploy keys      |       |          |           | ✓      | ✓     |
| 配置hooks            |       |          |           | ✓      | ✓     |
| 切换visibility level |       |          |           |        | ✓     |
| 切换工程namespace    |       |          |           |        | ✓     |
| 移除工程             |       |          |           |        | ✓     |
| 强制提交保护分支     |       |          |           |        | ✓     |
| 移除保护分支         |       |          |           |        | ✓     |

**2.组权限**

| 行为       | Guest | Reporter | Developer | Master | Owner |
| ---------- | ----- | -------- | --------- | ------ | ----- |
| 浏览组     | ✓     | ✓        | ✓         | ✓      | ✓     |
| 编辑组     |       |          |           |        | ✓     |
| 创建项目   |       |          |           | ✓      | ✓     |
| 下载工程   |       |          |           |        | ✓     |
| 管理组成员 |       |          |           |        | ✓     |
| 移除组     |       |          |           |        | ✓     |

# 五、分区

工作区（Working Directory） 是直接编辑的地方，肉眼可见，直接操作。

暂存区（Stage 或 Index） 数据暂时存放的区域。

版本库（commit History） 存放已经提交的数据，push 的时候，就是把这个区的数据 push 到远程git仓库了。

![image-20230113191423839](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202301131914298.png)

# 六、切换分支

如果本地有代码更新，并且没有提交暂存区的内容的时候，切换分支的时候，在新的分支上仍然能看到 新增的代码

`git checkout -b new_branch` 从当前分支上切换新的分支



# 八、使用步骤

1. 首先使用**git commit** ，保证自己写的代码保存起来
2. 使用git  fetch 获取远程修改的
3. 使用git pull 指令将远程修改的合并过来
4. 有冲突，手动解决冲突，之后再提交。

# 九、git指令

## git merge

如果另一个分支是当前提交的祖父节点，那么合并命令将什么也不做。 另一种情况是如果当前提交是另一个分支的祖父节点，就导致*fast-forward*合并。指向只是简单的移动，并生成一个新的提交。

<img src="https://www.runoob.com/wp-content/uploads/2019/05/1557394767-3629-merge-ff.svg-.png" alt="img" style="zoom:50%;" />就是一次真正的默认把当前提交(*ed489* 如下所示)和另一个提交(*33104*)以及他们的共同祖父节点(*b325c*)进行一次[三方合并](http://en.wikipedia.org/wiki/Three-way_merge)。结果是先保存当前目录和索引，然后和父节点*33104*一起做一次新提交。三方合并： 它会把两个分支的最新快照（`C3` 和 `C4`）以及二者最近的共同祖先（`C2`）进行三方合并，合并的结果是生成一个新的快照（并提交)。**就会导致棱形结构**

![img](https://www.runoob.com/wp-content/uploads/2019/05/1557394768-9049-merge.svg-.png)





## git rebase

衍合在当前分支上重演另一个分支的历史，提交历史是线性的,

上面的命令都在*topic*分支中进行，而不是*master*分支，在*master*分支上重演，并且把分支指向新的节点。注意旧提交没有被引用，将被回收。

<img src="https://www.runoob.com/wp-content/uploads/2019/05/1557394769-8202-rebase.svg-.png" alt="img" style="zoom:90%;" />

`git commit`执行时，会提交`暂存区`的内容到本地库；

 `git add` 命令会将我们做的修改`添加到暂存区`中。只有加入过一次，目录就一直处于暂存区。

`git diff `可以查看这次修改的内容（工作区 vs 暂存区）

`git diff head` 工作区 vs 版本库



如果远程从github获取应用的时候出现出现

Failed to connect to github.com port 443: Operation timed out

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220713202526299.png" alt="image-20220713202526299" style="zoom:50%;" />

可以使用这个命令

```sh
git config --global http.sslBackend "openssl"
```





Unable to negotiate with 123.57.204.47 port 22: no matching host key type found. Their offer: ssh-rsa,ssh-dss
fatal: Could not read from remote repository.

需要在~/.ssh/config 中添加命令

```
Host * 
HostKeyAlgorithms = +ssh-rsa
PubkeyAcceptedAlgorithms = +ssh-rsa
```

1. 
