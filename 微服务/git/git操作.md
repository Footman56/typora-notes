# idea 使用git

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410210051156.png" alt="image-20220422151123745" style="zoom:50%;" />

Local Changes : 展示本次未提交时的变化

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220422151307075.png" alt="image-20220422151307075" style="zoom:50%;" />

Log: 展示提交记录

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220422151547496.png" alt="image-20220422151547496" style="zoom:50%;" />

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410210051202.png" alt="image-20220422151729917" style="zoom:50%;" />

## 回退代码

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220422152013940.png" alt="image-20220422152013940" style="zoom:50%;" />

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220422152051631.png" alt="image-20220422152051631" style="zoom:50%;" />

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410210051824.png" alt="image-20220422152339054" style="zoom:50%;" />

Soft:  保留工作区及暂存区的修改

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220422153257176.png" alt="image-20220422153257176" style="zoom:50%;" />

此时还能看到项目中变化的提交的记录



Mixed： 保留工作区修改，清除暂存区

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410210051081.png" alt="image-20220422153817780" style="zoom:50%;" />

**可见Mixed 、Soft 效果是一样**



Hard: 不保留修改，直接回退到所选版本

```
git reset --hard xxxx(要回退的版本)
```

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410210051180.png" alt="image-20220422160338792" style="zoom:50%;" />



`git stash ` 将这次修改的 代码放到暂存区



**GitHub  升级啦。不在使用简单的 rsa做验证的**

1. 首先配置.git  里面的配置，设置姓名，邮件

2. 创建ssh key

   ```sh
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```

生成之后在让选择生成目录、密钥，直接跳过就可以了,

执行完命令之后在.ssh 目录里面 会有id_ed25519、id_ed25519.pub 这两个文件，id_ed25519.pub记录的是公钥，要放在github中。

推测：

![image-20230911113408979](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410210053771.png)

note: 默认只读取 id_rsa，为了让 SSH 识别新的私钥，需要将新的私钥加入到 ssh-agent 中

3. 

   ```\
   ssh-agent bash
   ssh-add ~/.ssh/id_ed25519
   ```

4. 将id_ed25519.pub里面的内容加入的gitbub中

5. 测试链接是否有效

   ```
   ssh -vT git@github.com
   ```

   ![image-20230911113953864](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202309111139928.png)

6. 如果在连接的过程中提示：

   ```
   Unable to negotiate with 123.57.204.47 port 22: no matching host key type found. Their offer: ssh-rsa,ssh-dss
   fatal: Could not read from remote repository.
   
   Please make sure you have the correct access rights
   and the repository exists.
   ```

    8.8p1 版的 openssh 的 ssh 客户端默认禁用了 `ssh-rsa` 算法, 但是对方服务器只支持 `ssh-rsa`, 当你不能自己升级远程服务器的 openssh 版本或修改配置让它使用更安全的算法时, 在本地 ssh 针对这些旧的ssh server重新启用 `ssh-rsa` 也是一种权宜之法.

​		在~/.ssh/config 文件下添加如下内容:		

```
Host ssh.alanwei.com
HostName ssh.alanwei.com
  User alan
  Port 22
  HostKeyAlgorithms +ssh-rsa
  PubkeyAcceptedKeyTypes +ssh-rsa
```

​	Host、HostName、User、Port可不填，







多账号配置

根据邮箱生成不同的ke y

```
ssh-keygen -t rsa -C 'yourEmail@xx.com' -f ~/.ssh/gitlab-rsa

ssh-keygen -t rsa -C 'yourEmail2@xx.com' -f ~/.ssh/github-rsa
```

```
# gitlab
Host gitlab.com
    HostName gitlab.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/gitlab_id-rsa
# github
Host github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/github_id-rsa
# 配置文件参数
# Host : Host可以看作是一个你要识别的模式，对识别的模式，进行配置对应的的主机名和ssh文件
# HostName : 要登录主机的主机名
# User : 登录名
# IdentityFile : 指明上面User对应的identityFile路径
```

注： github 放弃了rsa 验证方法，所有加密生成的key 不能使用这个算法，要使用ed25519

![image-20240527011006147](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410210053993.png)

出现上述描述是因为在配置文件中使用了 HostKeyAlgorithms配置，需要去掉，或则添加ed25519算法

查看远程仓库位置

```sh
git remote -v 
```





已经推送（push）过的文件，想从git远程库中删除，并在以后的提交中忽略，但是却还想在本地保留这个文件

```
git rm --cached 
```









# git 连接远程仓库

```sh
# 进入目录
git init

# 添加远程仓库 不要使用http地址，否则的话还需要输入账户密码
# git remote add origin https://github.com/Footman56/AutoGPT.git
git remote add origin git@github.com:Footman56/json-change.git
 
 
# 验证仓库地址
git remote -v


# 提交
git add.
git commit -m ""

# 推送
git push  -u origin main
```



# 忽略某个文件

git 强制忽略某个文件，要求切换分支时不影响

1. 在.gitignore 中添加需要忽略的路径

2. 如果已经提交到git上是，需要从git中删除

   ```bash
   git rm --cached /path/to/your/file
   
   # 如果是目录的话，
   git rm  -r --cached  /path
   ```

   
