# docker 形式安装one-api

```
docker run --name one-api -d --restart always -p 3000:3000 -e SQL_DSN="huochai:asdf1234@tcp(123.57.235.173:3306)/oneapi" -e TZ=Asia/Shanghai -v /home/huochai/one-api:/data ghcr.io/songquanpeng/one-api
```

在配置SQL_DSN 时不能指定localhost 因为是容器，而容器中没有数据库服务，使用的是服务器上的mysql 服务

访问  123.57.235.173:3000  进入页面

```
docker run --name one-api -d --restart always -p 3000:3000 -e SQL_DSN="huochai:asdf1234@tcp(123.57.235.173:3306)/oneapi" -e TZ=Asia/Shanghai  justsong/one-api
```







sk-c08da21ae0174d6fa4d86561afd3d91c







```
现在需要一个工具，工具的作用是根据给出的中文描述生成变量名、git分支名。变量名称要求符合java名称规范，并且可以选择出给选择命名格式，比如驼峰式，全部大写形式，变量名称包括普通变量、函数名称、常量、枚举名称、mysql数据库字段名称。系统是mac，java常用开发工具是idea，mac中有raycast、python3.12、brew。 请使用现有的工具未完成功能开发。
```

