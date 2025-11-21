安装除操作系统自带python 

# 下载、解压

从官网下载，并解压

```
wget https://www.python.org/ftp/python/3.12.3/Python-3.12.3.tar.xz

tar -xf Python-3.12.3.tar.xz
```

# 安装必要的依赖

````shell
sudo yum -y groupinstall "Development Tools"
sudo yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel libffi-devel gcc-c++
````

# 清理之前的构造

```sh
make clean
```

# 配置

```shell
./configure  --prefix=/usr/local/python3.12
```

# 编译

```
make
```

# 安装

```shell
sudo make altinstall
```

# 检查

```
python3.12 --version
```



# 创建虚拟环境

先安装  venv

在当前目录下创建虚拟环境

```sh
python3. -m venv myenv
```

# 激活虚拟环境

```
source myenv/bin/activate
```

宗旨：在虚拟环境使用python3 、 pip3 

后续可以正常下载包

![image-20241020000349554](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410200003632.png)

# 后台运行程序

```sh
nohup  python3 ap.py &
```









# pycharm

1. 下载破解插件

   https://pan.baidu.com/s/1eU7uYJ_DDMMYeFvn8HCJbg?pwd=euoa   激活码：euoa

2. 解压，目录不能有中文

3. 执行install.sh, 目录在`/Users/peilizhi/pythonProjects/jetBra/scripts`

4. 激活码

   ```
   EUWT4EE9X2-eyJsaWNlbnNlSWQiOiJFVVdUNEVFOVgyIiwibGljZW5zZWVOYW1lIjoic2lnbnVwIHNjb290ZXIiLCJhc3NpZ25lZU5hbWUiOiIiLCJhc3NpZ25lZUVtYWlsIjoiIiwibGljZW5zZVJlc3RyaWN0aW9uIjoiIiwiY2hlY2tDb25jdXJyZW50VXNlIjpmYWxzZSwicHJvZHVjdHMiOlt7ImNvZGUiOiJQU0kiLCJmYWxsYmFja0RhdGUiOiIyMDI1LTA4LTAxIiwicGFpZFVwVG8iOiIyMDI1LTA4LTAxIiwiZXh0ZW5kZWQiOnRydWV9LHsiY29kZSI6IlBDIiwiZmFsbGJhY2tEYXRlIjoiMjAyNS0wOC0wMSIsInBhaWRVcFRvIjoiMjAyNS0wOC0wMSIsImV4dGVuZGVkIjpmYWxzZX0seyJjb2RlIjoiUFBDIiwiZmFsbGJhY2tEYXRlIjoiMjAyNS0wOC0wMSIsInBhaWRVcFRvIjoiMjAyNS0wOC0wMSIsImV4dGVuZGVkIjp0cnVlfSx7ImNvZGUiOiJQV1MiLCJmYWxsYmFja0RhdGUiOiIyMDI1LTA4LTAxIiwicGFpZFVwVG8iOiIyMDI1LTA4LTAxIiwiZXh0ZW5kZWQiOnRydWV9LHsiY29kZSI6IlBDV01QIiwiZmFsbGJhY2tEYXRlIjoiMjAyNS0wOC0wMSIsInBhaWRVcFRvIjoiMjAyNS0wOC0wMSIsImV4dGVuZGVkIjp0cnVlfV0sIm1ldGFkYXRhIjoiMDEyMDIyMDkwMlBTQU4wMDAwMDUiLCJoYXNoIjoiVFJJQUw6MzUzOTQ0NTE3IiwiZ3JhY2VQZXJpb2REYXlzIjo3LCJhdXRvUHJvbG9uZ2F0ZWQiOmZhbHNlLCJpc0F1dG9Qcm9sb25nYXRlZCI6ZmFsc2V9-FT9l1nyyF9EyNmlelrLP9rGtugZ6sEs3CkYIKqGgSi608LIamge623nLLjI8f6O4EdbCfjJcPXLxklUe1O/5ASO3JnbPFUBYUEebCWZPgPfIdjw7hfA1PsGUdw1SBvh4BEWCMVVJWVtc9ktE+gQ8ldugYjXs0s34xaWjjfolJn2V4f4lnnCv0pikF7Ig/Bsyd/8bsySBJ54Uy9dkEsBUFJzqYSfR7Z/xsrACGFgq96ZsifnAnnOvfGbRX8Q8IIu0zDbNh7smxOwrz2odmL72UaU51A5YaOcPSXRM9uyqCnSp/ENLzkQa/B9RNO+VA7kCsj3MlJWJp5Sotn5spyV+gA==-MIIETDCCAjSgAwIBAgIBDTANBgkqhkiG9w0BAQsFADAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBMB4XDTIwMTAxOTA5MDU1M1oXDTIyMTAyMTA5MDU1M1owHzEdMBsGA1UEAwwUcHJvZDJ5LWZyb20tMjAyMDEwMTkwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCUlaUFc1wf+CfY9wzFWEL2euKQ5nswqb57V8QZG7d7RoR6rwYUIXseTOAFq210oMEe++LCjzKDuqwDfsyhgDNTgZBPAaC4vUU2oy+XR+Fq8nBixWIsH668HeOnRK6RRhsr0rJzRB95aZ3EAPzBuQ2qPaNGm17pAX0Rd6MPRgjp75IWwI9eA6aMEdPQEVN7uyOtM5zSsjoj79Lbu1fjShOnQZuJcsV8tqnayeFkNzv2LTOlofU/Tbx502Ro073gGjoeRzNvrynAP03pL486P3KCAyiNPhDs2z8/COMrxRlZW5mfzo0xsK0dQGNH3UoG/9RVwHG4eS8LFpMTR9oetHZBAgMBAAGjgZkwgZYwCQYDVR0TBAIwADAdBgNVHQ4EFgQUJNoRIpb1hUHAk0foMSNM9MCEAv8wSAYDVR0jBEEwP4AUo562SGdCEjZBvW3gubSgUouX8bOhHKQaMBgxFjAUBgNVBAMMDUpldFByb2ZpbGUgQ0GCCQDSbLGDsoN54TATBgNVHSUEDDAKBggrBgEFBQcDATALBgNVHQ8EBAMCBaAwDQYJKoZIhvcNAQELBQADggIBABqRoNGxAQct9dQUFK8xqhiZaYPd30TlmCmSAaGJ0eBpvkVeqA2jGYhAQRqFiAlFC63JKvWvRZO1iRuWCEfUMkdqQ9VQPXziE/BlsOIgrL6RlJfuFcEZ8TK3syIfIGQZNCxYhLLUuet2HE6LJYPQ5c0jH4kDooRpcVZ4rBxNwddpctUO2te9UU5/FjhioZQsPvd92qOTsV+8Cyl2fvNhNKD1Uu9ff5AkVIQn4JU23ozdB/R5oUlebwaTE6WZNBs+TA/qPj+5/we9NH71WRB0hqUoLI2AKKyiPw++FtN4Su1vsdDlrAzDj9ILjpjJKA1ImuVcG329/WTYIKysZ1CWK3zATg9BeCUPAV1pQy8ToXOq+RSYen6winZ2OO93eyHv2Iw5kbn1dqfBw1BuTE29V2FJKicJSu8iEOpfoafwJISXmz1wnnWL3V/0NxTulfWsXugOoLfv0ZIBP1xH9kmf22jjQ2JiHhQZP7ZDsreRrOeIQ/c4yR8IQvMLfC0WKQqrHu5ZzXTH4NO3CwGWSlTY74kE91zXB5mwWAx1jig+UXYc2w4RkVhy0//lOmVya/PEepuuTTI4+UJwC7qbVlh5zfhj8oTNUXgN0AOc+Q0/WFPl1aw5VV/VrO8FCoB15lFVlpKaQ1Yh+DVU8ke+rt9Th0BCHXe0uZOEmH0nOnH/0onD
   ```


通过pycharm 创建项目的时候 已经默认生成 .venv 

下载的时候只需选择 .venv

```
 python3 -m  pip install  openai 
```















```
GAJWL09BT5RSXDR-eyJsaWNlbnNlSWQiOiJHQUpXTDA5QlQ1UlNYRFIiLCJsaWNlbnNlZU5hbWUiOiJtZW5vcmFoIHBhcmFwZXQiLCJsaWNlbnNlZVR5cGUiOiJQRVJTT05BTCIsImFzc2lnbmVlTmFtZSI6IiIsImFzc2lnbmVlRW1haWwiOiIiLCJsaWNlbnNlUmVzdHJpY3Rpb24iOiIiLCJjaGVja0NvbmN1cnJlbnRVc2UiOmZhbHNlLCJwcm9kdWN0cyI6W3siY29kZSI6IlBDV01QIiwiZmFsbGJhY2tEYXRlIjoiMjAyNi0wOS0xNCIsInBhaWRVcFRvIjoiMjAyNi0wOS0xNCIsImV4dGVuZGVkIjp0cnVlfSx7ImNvZGUiOiJQQyIsImZhbGxiYWNrRGF0ZSI6IjIwMjYtMDktMTQiLCJwYWlkVXBUbyI6IjIwMjYtMDktMTQiLCJleHRlbmRlZCI6ZmFsc2V9LHsiY29kZSI6IlBTSSIsImZhbGxiYWNrRGF0ZSI6IjIwMjYtMDktMTQiLCJwYWlkVXBUbyI6IjIwMjYtMDktMTQiLCJleHRlbmRlZCI6dHJ1ZX1dLCJtZXRhZGF0YSI6IjAyMjAyNDA3MDJQU0FYMDAwMDA1WCIsImhhc2giOiIxMjM0NTY3OC8wLTQ2MTc4NjQwOSIsImdyYWNlUGVyaW9kRGF5cyI6NywiYXV0b1Byb2xvbmdhdGVkIjpmYWxzZSwiaXNBdXRvUHJvbG9uZ2F0ZWQiOmZhbHNlLCJ0cmlhbCI6ZmFsc2UsImFpQWxsb3dlZCI6dHJ1ZX0=-UeOCCiS72PGvOIS9go0yIhDFVmPBvbKM56D9w0adVaGcYLtC7YxNr/5MQ/3+Mr05tQQAhMz12vBTb9sjJAXBo+HBzCv1o9IFZnJK2rf3pCXl83ulriBUQ6M0H6GUUy+Mc1fl0EGWquoNExZMujCkReWoeabxwwKPNCvHqHqkW1rU/+cwiVKjVfbIgQW9aChIwyYwexzSlM0TlHvQGfncEzI0+uYNxjRQUjemLlGJooYD0ycSMMTyTvM95QHi25DZjmQRkdzIhDA2l4uPp+C+XEAIdIST2rjEPolvJGcVu7P/DI77LDDqZwLtD8mFXh9lFqMEw9titvy4mYFlYp/xaw==-MIIETDCCAjSgAwIBAgIBDTANBgkqhkiG9w0BAQsFADAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBMB4XDTIwMTAxOTA5MDU1M1oXDTIyMTAyMTA5MDU1M1owHzEdMBsGA1UEAwwUcHJvZDJ5LWZyb20tMjAyMDEwMTkwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCUlaUFc1wf+CfY9wzFWEL2euKQ5nswqb57V8QZG7d7RoR6rwYUIXseTOAFq210oMEe++LCjzKDuqwDfsyhgDNTgZBPAaC4vUU2oy+XR+Fq8nBixWIsH668HeOnRK6RRhsr0rJzRB95aZ3EAPzBuQ2qPaNGm17pAX0Rd6MPRgjp75IWwI9eA6aMEdPQEVN7uyOtM5zSsjoj79Lbu1fjShOnQZuJcsV8tqnayeFkNzv2LTOlofU/Tbx502Ro073gGjoeRzNvrynAP03pL486P3KCAyiNPhDs2z8/COMrxRlZW5mfzo0xsK0dQGNH3UoG/9RVwHG4eS8LFpMTR9oetHZBAgMBAAGjgZkwgZYwCQYDVR0TBAIwADAdBgNVHQ4EFgQUJNoRIpb1hUHAk0foMSNM9MCEAv8wSAYDVR0jBEEwP4AUo562SGdCEjZBvW3gubSgUouX8bOhHKQaMBgxFjAUBgNVBAMMDUpldFByb2ZpbGUgQ0GCCQDSbLGDsoN54TATBgNVHSUEDDAKBggrBgEFBQcDATALBgNVHQ8EBAMCBaAwDQYJKoZIhvcNAQELBQADggIBABKaDfYJk51mtYwUFK8xqhiZaYPd30TlmCmSAaGJ0eBpvkVeqA2jGYhAQRqFiAlFC63JKvWvRZO1iRuWCEfUMkdqQ9VQPXziE/BlsOIgrL6RlJfuFcEZ8TK3syIfIGQZNCxYhLLUuet2HE6LJYPQ5c0jH4kDooRpcVZ4rBxNwddpctUO2te9UU5/FjhioZQsPvd92qOTsV+8Cyl2fvNhNKD1Uu9ff5AkVIQn4JU23ozdB/R5oUlebwaTE6WZNBs+TA/qPj+5/we9NH71WRB0hqUoLI2AKKyiPw++FtN4Su1vsdDlrAzDj9ILjpjJKA1ImuVcG329/WTYIKysZ1CWK3zATg9BeCUPAV1pQy8ToXOq+RSYen6winZ2OO93eyHv2Iw5kbn1dqfBw1BuTE29V2FJKicJSu8iEOpfoafwJISXmz1wnnWL3V/0NxTulfWsXugOoLfv0ZIBP1xH9kmf22jjQ2JiHhQZP7ZDsreRrOeIQ/c4yR8IQvMLfC0WKQqrHu5ZzXTH4NO3CwGWSlTY74kE91zXB5mwWAx1jig+UXYc2w4RkVhy0//lOmVya/PEepuuTTI4+UJwC7qbVlh5zfhj8oTNUXgN0AOc+Q0/WFPl1aw5VV/VrO8FCoB15lFVlpKaQ1Yh+DVU8ke+rt9Th0BCHXe0uZOEmH0nOnH/0onD
```









# 卸载python脚本

```sh
#!/bin/bash
# clean_user_python.sh - macOS彻底清理用户安装的 Python，保留系统自带 Python (/usr/bin/python3)

echo "⚠️ 开始彻底清理用户安装的 Python 和相关工具（保留 /usr/bin/python3）..."

# 1️⃣ 卸载 Homebrew 安装的 Python
if command -v brew >/dev/null 2>&1; then
  echo "💧 卸载 Homebrew Python..."
  brew list | grep python | xargs -I {} brew uninstall --ignore-dependencies {} 2>/dev/null
fi

# 2️⃣ 删除 Homebrew Python 残留文件
echo "💧 删除 Homebrew 残留文件..."
sudo rm -rf /usr/local/bin/python* /usr/local/bin/pip* /usr/local/bin/jupyter* /usr/local/lib/python* /usr/local/include/python*
sudo rm -rf /opt/homebrew/bin/python* /opt/homebrew/bin/pip* /opt/homebrew/bin/jupyter* /opt/homebrew/lib/python* /opt/homebrew/include/python*

# 3️⃣ 删除 Anaconda / Miniconda
echo "💧 删除 Anaconda / Miniconda..."
rm -rf /opt/anaconda3 ~/opt/anaconda3 ~/.conda ~/.condarc ~/.continuum ~/.anaconda_backup

# 4️⃣ 删除用户目录下 Python 包、虚拟环境和缓存
echo "💧 删除用户目录 Python 包、虚拟环境和缓存..."
rm -rf ~/Library/Python ~/.local/lib/python* ~/.cache/pip ~/Library/Caches/pip ~/venvs ~/envs ~/bin/python*

# 5️⃣ 删除 shell 配置中 Anaconda / Python PATH
echo "💧 清理 shell 配置..."
for file in ~/.zshrc ~/.bash_profile ~/.bashrc ~/.profile; do
  [ -f "$file" ] && sed -i.bak "/\/opt\/anaconda3/d" "$file"
done

# 6️⃣ 清理 Homebrew 缓存和残留
if command -v brew >/dev/null 2>&1; then
  echo "💧 清理 Homebrew 缓存..."
  brew cleanup -s
  brew doctor
fi

# 7️⃣ 输出检查结果
echo "✅ 清理完成，仅保留系统自带 Python (/usr/bin/python3)"
echo "which python3: $(which python3)"
echo "which python: $(which python || echo not found)"
echo "which pip: $(which pip || echo not found)"
echo "which conda: $(which conda || echo not found)"
echo "which jupyter: $(which jupyter || echo not found)"
```



# 将python3 --version指向用户下载的版本

```
1. 首先检查用户版本python的安装位置
brew --prefix python@3.13


2. 进入对应目录，检查可执行程序的名称

3. 在.bash_profile中添加
alias python3="/usr/local/opt/python@3.13/bin/python3"
alias pip3="/usr/local/opt/python@3.13/bin/pip3"

4. 刷新.bash_profile
```

