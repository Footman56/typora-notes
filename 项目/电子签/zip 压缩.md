```xml
<dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-compress</artifactId>
            <version>1.19</version>
        </dependency>
```

```java
/**
     * 下载PDF文件并打包成压缩包
     * @param assessExportSignDOS
     * @return 压缩包文件
     */
    public File downloadAndZipPdfs(List<KpiAssessExportSignDO> assessExportSignDOS, String zipFileName) {
        File zipFile = null;
        StopWatch stopWatch = new StopWatch();
        stopWatch.start("批量压缩问题");
        // 压缩根目录,
        String parentsPath = System.getProperty("user.dir") + "/appraisalContractFile/";
        File appraisalContractPath = new File(parentsPath);
        if (!appraisalContractPath.exists()) {
          // 创建的时候需要保证有权限，否则会提示创建失败
            appraisalContractPath.mkdirs();
        }
        // 获取zip文件名称-支持中文
        String fileName = zipFileName.replaceAll(".zip", "");

        // 压缩临时目录-用于存储所有需要压缩的文件
        String tempZipPath =System.getProperty("user.dir") +  "/appraisalContractFile/" + fileName + "/";
        File tempZipPathDic = new File(tempZipPath);
        if (!tempZipPathDic.exists()) {
            tempZipPathDic.mkdirs();
        }

        // 收集
        List<File> pdfFiles = new ArrayList<>();
        // 下载pdf文件
        for (KpiAssessExportSignDO signDO : assessExportSignDOS) {
            String downloadUrl = signDO.getDownloadUrl();
            String pdfFileName = signDO.getAssessEmpName() + "_" + signDO.getContractName() + ".pdf";
            pdfFileName = new String(pdfFileName.getBytes(), StandardCharsets.UTF_8);
            try {
                File pdfFile = new File(tempZipPath, pdfFileName);
                log.info("downloadAndZipPdfs downloadUrl={}, pdfFileName={}", downloadUrl, pdfFileName);
                downloadFile(downloadUrl, pdfFile);
                pdfFiles.add(pdfFile);
            } catch (IOException e) {
                log.info("downloadAndZipPdfs 下载文件异常 ", e);
            }
        }
        try {
            zipFile = new File(tempZipPathDic.getParentFile(), zipFileName);
            zipFiles(pdfFiles, fileName, zipFile);
        } catch (Exception e) {
            log.info("批量下载合同文件异常 ", e);
            throw new XrxsException(EAppraisalErrorCode.ELECTRONIC_CONTRACT_DOWNLOAD_FAIL);
        } finally {
            if (CollectionUtils.isNotEmpty(pdfFiles)){
                for (File pdfFile : pdfFiles) {
                    pdfFile.delete();
                }
            }
            tempZipPathDic.delete();
            stopWatch.stop();
            log.info("批量下载合同文件总耗时：{}(ms)", stopWatch.prettyPrint());
        }
        return zipFile;
    }


    /**
     * 下载文件
     * @param downloadUrl 下载链接
     * @param destFile 目标文件
     * @throws IOException
     */
    private void downloadFile(String downloadUrl, File destFile) throws IOException {
        try {
            // 创建 URL 对象
            URL url = new URL(downloadUrl);
            // 打开连接
            URLConnection connection = url.openConnection();
            connection.connect();

            // 获取文件名
            // 创建输入流读取下载内容
            try (InputStream inputStream = connection.getInputStream();
                 BufferedInputStream bufferedInputStream = new BufferedInputStream(inputStream);
                 FileOutputStream fileOutputStream = new FileOutputStream(destFile);
                 BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(fileOutputStream)) {

                // 缓冲区用于提高下载效率
                byte[] buffer = new byte[4096];
                int bytesRead;

                // 循环读取数据并写入文件
                while ((bytesRead = bufferedInputStream.read(buffer)) != -1) {
                    bufferedOutputStream.write(buffer, 0, bytesRead);
                }
            }
        } catch (IOException e) {
            log.error("下载文件失败 downloadUrl:{}", downloadUrl, e);
        }
    }


    /**
     * 打包文件夹中的所有文件成压缩包
     * @param sourceDir 源文件夹
     * @param zipFile 压缩包文件
     * @throws IOException
     */
    private void zipFiles(List<File> files, String parentDir, File zipFile) throws IOException {
        try (FileOutputStream fos = new FileOutputStream(zipFile);
             ZipArchiveOutputStream zos = new ZipArchiveOutputStream(fos)) {
            // 设置压缩包编码
            zos.setEncoding("UTF-8");
            if (CollectionUtils.isNotEmpty(files)) {
                for (File file : files) {
                    addToZipFile(file, parentDir, zos);
                }
            }
        }
    }


    /**
     * 将文件添加到压缩包中
     * @param file 文件
     * @param parentDir 父目录名称
     * @param zos 压缩包输出流
     * @throws IOException
     */
    private void addToZipFile(File file, String parentDir, ZipArchiveOutputStream zos) throws IOException {
       // 注：zipName 如果是 tmp/appraisal.pdf 的话 那么解压后的就是 tmp目录下有appraisal.pdf
        String zipEntryName = parentDir + File.separator + file.getName();
        ZipArchiveEntry zipEntry = new ZipArchiveEntry(file, zipEntryName);
        zos.putArchiveEntry(zipEntry);

        IOUtils.copy(new FileInputStream(file), zos);
        zos.closeArchiveEntry();
    }
```



```
File pdfFile = new File(tempZipPath, pdfFileName);
是在tempZipPath目录下创建pdfFileName的文件,pdfFileName 是中文名称，pdfFile显示的文件是正确的中文
但是tempZipPath.listFiles()时显示的文件名称是乱码

解决方式使用生成好的pdfFile收集到一个集合里面、不要使用listFiles()
```



解决生成文件乱码

1、检查服务器语言

```sh
locale
```

<img src="https://cdn.jsdelivr.net/gh/Footman56/imageBed2/202501231837594.png" alt="image-20250123183717389" style="zoom:50%;" />

en_US 、zh_US 都支持中文

```
export LANG=zh_CN.UTF-8
export LC_ALL=en_US.UTF-8
localedef -i en_US -f UTF-8 en_US.UTF-8
```

2、检查程序是否设置UTF-8

```
-Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8
```

3、 前两步设置正确后基本就可以啦，如果还是不行的话，就修改文件名的编码





