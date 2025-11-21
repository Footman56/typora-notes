**mac的一些指令很多情况都可以参考Linux的指令**

# 用户、用户组

查看用户组：

```
dscl . list /groups
```

 查看用户：

```
dscl . list /users
```

 添加用户组：

```
sudo dscl . -create /Groups/test
```

 删除用户组：

```
sudo dscl . -delete /Groups/test
```

 添加用户：

```
sudo dscl .  -create /Users/redis
```

 删除用户：

```
sudo dscl . -delete /Users/redis
```

 显示所有users对应的group:

```
sudo dscl . -list /groups GroupMembership 
```

 添加user到group:

```
sudo dscl . -append /Groups/groupname GroupMembership username 
```

从group中删除user：

```
sudo dscl . -delete /Groups/groupname GroupMembership username
```

查看端口号

```shell
#列出所有端口
netstat 

#列出所有 tcp 端口 
netstat -at

#显示网络接口列表
netstat -i
```

查看某一端口占用情况



```shell
#命令格式：lsof -i :端口
lsof -i:8080
```

