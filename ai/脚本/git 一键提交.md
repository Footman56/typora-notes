1. 安装 GitHub CLI：

   ```
   brew install gh   # Mac
   ```

2. 需要登录 GitHub CLI

   ```
   1. 选择https
   
   2. 输入对应的token 
   ```

   ![image-20250905112245147](https://raw.githubusercontent.com/Footman56/images/master/img202509051122647.png)

3. 安装 Copilot 插件：

   ```
   gh extension install github/gh-copilot
   ```

   





```
 DIFF=$(git diff --cached)
$(echo "$DIFF" | gh copilot suggest -t shell "Generate a concise git commit message for these changes")
```





