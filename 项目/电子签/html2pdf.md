目的实现html文件转化成pdf文件，需要重点关注html中的css兼容性问题、字体问题、图片问题、html内脚本问题

样式问题主要集中在 `display: flex;`

# html2pdf

html2pdf 是jar ，可以直接讲html字符串转化成pdf

1. pom配置

   ```xml
   <dependency>
               <groupId>com.itextpdf</groupId>
               <artifactId>html2pdf</artifactId>
               <version>6.0.0</version>
   </dependency>
   ```

2. 使用

   ```java
    public void generatePdf(HttpServletResponse response, String htmlContent) throws IOException, DocumentException {
           // 设置响应类型
           response.setContentType("application/pdf");
           response.setHeader("Content-Disposition", "attachment; filename=generated.pdf");
   
           ConverterProperties pro = new ConverterProperties();
           FontProvider fontProvider = new DefaultFontProvider();
           fontProvider.addSystemFonts();
   
           
           // 设置字体路径，使用 classpath 加载字体
           ClassPathResource dDinFont = new ClassPathResource("/templates/fonts/D-DIN.ttf");
           FontProgram dDinFontProgram = FontProgramFactory.createFont(dDinFont.getFile().getAbsolutePath());
           fontProvider.addFont(dDinFontProgram);
           
           ClassPathResource dinProFont = new ClassPathResource("/templates/fonts/DINPro.ttf");
           FontProgram dinProProgram = FontProgramFactory.createFont(dinProFont.getFile().getAbsolutePath());
           fontProvider.addFont(dinProProgram);
   
           ClassPathResource pinFangFont = new ClassPathResource("/templates/fonts/PingFang SC.ttf");
           FontProgram pinFangProgram = FontProgramFactory.createFont(pinFangFont.getFile().getAbsolutePath());
           fontProvider.addFont(pinFangProgram);
           pro.setFontProvider(fontProvider);
           
           // 输出PDF到响应输出流
           try (OutputStream outputStream = response.getOutputStream()) {
               PageSize pageSize = PageSize.A4;
               PdfWriter pdfWriter = new PdfWriter(outputStream);
               PdfDocument pdfDocument = new PdfDocument(pdfWriter);
               pdfDocument.setDefaultPageSize(pageSize);
               HtmlConverter.convertToPdf(htmlContent, pdfWriter, pro);
               outputStream.flush();
           }
       }
   ```

**注：这个转化的时候也是有样式问题**  

# wkhtmltopdf

这个一个app ,  需要下载。 下载后就可以使用直接转化html 到pdf 

```shell
wkhtmltopdf --encoding utf-8 --no-images view.html test.pdf
```

**注：这个转化的时候也是有样式问题**  

# puppeteer

0. 需要node环节、puppeteer库、chrome

1. 配置node环节

   node版本新测为**v20.18.1** 可以

2. puppeteer

   puppeteer版本 为24.0.0 ，正常来说下载puppeteer时是会自动安装chrome的,但是因为国内环境限制无法下载， 所有下载时需要跳过chrome ，手动下载chrome，之后在脚本中指定自定义的chrome

   ```sh
   #下面这两种都会下载chrome
   yarn add puppeteer 
   npm i puppeteer
   
   # 可以试试忽略chrome
   yarn add puppeteer  PUPPETEER_SKIP_CHROMIUM_DOWNLOAD
   ```

   记录下载路径 `/usr/local/node/v20.18.1/lib/node_modules/puppeteer`

3. 手动下载chrome 路径

​		`/usr/local/chrome-linux/` 下载时不要指定到root路径，否则的话需要配置权限

4. 创建临时目录用于转化pdf

   ```sh
   cd /tmp
   mkdir  tempHtml2pdf
   ```

5. 创建js脚本

   ```sh
   cd /tmp/tempHtml2pdf
   # 创建名为convertHtmlToPdf.js 的脚本
   touch convertHtmlToPdf.js
   # 设置可执行权限
   chmod 755 convertHtmlToPdf.js
   # 创建测试页面
   touch view.html
   ```

   脚本

   `/usr/local/node/v20.18.1/lib/node_modules/puppeteer` 为puppeteer 安装路径

   `/usr/local/chrome-linux/chrome` 自定义chrome路径

   args: 使用的时候功能比较简单不需要浏览器复杂的功能，比如音效等

   ```js
   const fs = require('fs');
   const puppeteer = require('/usr/local/node/v20.18.1/lib/node_modules/puppeteer');
   
   async function convertHtmlToPdf(inputHtmlFile, outputPdfFile) {
       const browser = await puppeteer.launch({
                                                  executablePath: '/usr/local/chrome-linux/chrome',
                                                  headless:true,
                                                  args: ['--no-sandbox', '--disable-setuid-sandbox', '--disable-gpu', '--headless', '--disable-software-rasterizer', '--disable-extensions', '--mute-audio']
                                              });
       const page = await browser.newPage();
   
       const htmlContent = fs.readFileSync(inputHtmlFile, 'utf-8');
       await page.setContent(htmlContent);
       await page.pdf({
                          path: outputPdfFile,
                          format: 'A4',
                          printBackground: true
                      });
   
       await browser.close();
   }
   
   const inputHtmlFile = process.argv[2];
   const outputPdfFile = process.argv[3];
   
   convertHtmlToPdf(inputHtmlFile, outputPdfFile)
       .then(() => {
           console.log("PDF generated successfully!");
       })
       .catch((error) => {
           console.error("Error generating PDF:", error);
   });
   ```

   注实际情况中可能就算配置的args参数，时机执行过程中也会出现提示，此时就真的需要下载对应的依赖啦

   <img src="https://cdn.jsdelivr.net/gh/Footman56/imageBed2/202501231500806.png" alt="image-20250123150000751" style="zoom:43%;" />

   ```
   报错:/home/work/node_modules/puppeteer/.local-chromium/linux-856583/chrome-linux/chrome: error while loading shared libraries: libatk-1.0.so.0: cannot open shared object file: No such file or directory
   解决: yum install atk
   
   报错:/home/work/node_modules/puppeteer/.local-chromium/linux-856583/chrome-linux/chrome: error while loading shared libraries: libatk-bridge-2.0.so.0: cannot open shared object file: No such file or directory
   解决: yum install at-spi2-atk
   
   报错:/home/work/node_modules/puppeteer/.local-chromium/linux-856583/chrome-linux/chrome: error while loading shared libraries: libxkbcommon.so.0: cannot open shared object file: No such file or directory
   解决:yum install libxkbcommon-x11-devel
   
   报错:/home/work/node_modules/puppeteer/.local-chromium/linux-856583/chrome-linux/chrome: error while loading shared libraries: libXcomposite.so.1: cannot open shared object file: No such file or directory
   解决:yum install libXcomposite
   
   
   报错:/home/work/node_modules/puppeteer/.local-chromium/linux-856583/chrome-linux/chrome: error while loading shared libraries: libgtk-3.so.0: cannot open shared object file: No such file or directory
   解决:yum install gtk3	
   ```

   检查是否都安装完依赖

   ```
   # 进入chrome安装目录，使用下面命令检查是否都安装完依赖
   ldd chrome| grep not
   ```

6. 实际运行,注在运行指令的时候 view.html 、test.pdf 都需要有可操作权限

   ```sh
   node convertHtmlToPdf.js view.html test.pdf
   ```

7. 实际运行的时候需要保证linux环境中有中文字体

   ```
   # 安装中文字体
   1. 上传字体文件到/usr/share/fonts/chinese
   2. 生效字体 fc-cache -fv
   
   # 检查系统中安装的中文字体
   fc-list :lang=zh-cn
   # 配置文件在/etc/fonts/fonts.conf 
   ```

