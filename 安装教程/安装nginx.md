1. 安装nginx

   ```
   sudo yum install nginx
   ```

2. 启动nginx 服务

   ```
   sudo systemctl start nginx
   ```

3. 检查nginx 状态

   ```
   sudo systemctl status nginx
   ```

   ![image-20250130164302275](/Users/peilizhi/Library/Application Support/typora-user-images/image-20250130164302275.png)

4. 设置开机启动

   ```
   sudo systemctl enable nginx
   ```

   

5. 创建html文件

   ```
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>Welcome to nginx! -huochai</title>
   </head>
   <body>
       <h1>Welcome to nginx!</h1>
       <p>If you see this page, the nginx web server is successfully installed and working. Congratulations!</p>
   </body>
   </html>
   ```

   