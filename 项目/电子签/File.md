# 获取文件对象

`new File()并不是创建文件，而是一个抽象的文件对象，如果文件不存在的话，就无法使用`

1. 直接写文件名称，不指定目录

   此时创建路径为应用程序目标下面

```java
 // 创建文件
File file = new File("test.pdf");
String absolutePath = file.getAbsolutePath();
// /Users/peilizhi/javaProjects/spring-student/test.pdf
System.out.println("absolutePath = " + absolutePath);
```

2. 通过目录和文件名创建

   如果目录不存在的话，createNewFile 会提示：java.io.IOException: No such file or directory

```java
File file = new File("/Users/peilizhi/hh","test.pdf");
        String absolutePath = file.getAbsolutePath();
        System.out.println("absolutePath = " + absolutePath);

        boolean exists = file.exists();
        if (!exists){
            try {
                boolean newFile = file.createNewFile();
                System.out.println("newFile = " + newFile);
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }
        System.out.println("exists = " + exists);
```

3. 通过目录文件和文件名创建 同第二步

# 写入数据

1. 通过下载链接来写入内容

```java
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
```

2. Files.write()

   写入内容，并且内容是换行的

```java
// 文件路径
        String filePath = "output.txt"; // 可以是绝对路径或相对路径
        // 要写入的内容
        List<String> lines = new ArrayList<>();
        lines.add("Hello, World!");
        lines.add("This is a test.");
        lines.add("Have a nice day!");
        lines.add("Have a nice day!");
        lines.add("Have a nice day!");
        lines.add("这是一个中文");

        try {
            // 创建Path对象
            Path path = Paths.get(filePath);

            // 如果文件不存在，则创建文件
            if (!Files.exists(path)) {
                Files.createFile(path);
                System.out.println("文件已创建：" + path.toAbsolutePath());
            }

            // 写入内容到文件
            Files.write(path, lines);
            System.out.println("内容已写入文件：" + path.toAbsolutePath());
        } catch (IOException e) {
           e.printStackTrace();
        }
```

3. BufferedWriter 适合大文件，性能好，并且需要手动指定换行符

```java
// 文件路径
String filePath = "output.txt"; // 可以是绝对路径或相对路径

try (BufferedWriter writer = new BufferedWriter(new FileWriter(filePath))) {
  // 写入多行内容
  writer.write("第一行内容");
  writer.newLine(); // 写入换行符
  writer.write("第二行内容");
  writer.newLine();
  writer.write("第三行内容");

  System.out.println("内容已写入文件：" + filePath);
} catch (IOException e) {
  e.printStackTrace();
}
```

# 读取数据

1. 使用ClassPathResource 读取Resource 下目录

```java
try (InputStream inputStream = new ClassPathResource("file/预置指标库.xlsx").getInputStream()) {
  List<LinkedHashMap<String, String>> excelData = ImportExcelHelper.importExcel(inputStream, "预置指标库.xlsx");
  List<LinkedHashMap<String, String>> temp = new ArrayList<>(excelData.size());
  // 过滤空行
  excelData.forEach(data -> {
    // 过滤空行，并改变Excel表每行数据的key: header0--> 18900000000，header1--> 张三，...
    boolean allEmpty = true;
    for (Map.Entry<String, String> entry : data.entrySet()) {
      if (!StringUtil.isEmpty(entry.getValue())) {
        allEmpty = false;
      }
    }
    if (!allEmpty) {
      temp.add(data);
    }
  });
  content = temp;
  return content;
} catch (Exception e) {
  log.error("获取项目本地文件失败,文件名:预置指标库.xlsx,路径：file/预置指标库.xlsx");
  throw new XrxsException(EAppraisalErrorCode.KPI_PRESET_LIBRARY_EXCEL_FAILED);
}
```

2. 小文件 需要Apache Commons IO

```xml
<dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-io</artifactId>
 </dependency>
```

```java
import org.apache.commons.io.FileUtils;

import java.io.File;
import java.io.IOException;

public class ReadFileWithCommonsIO {
    public static void main(String[] args) {
        String filePath = "example.txt";
        try {
            // 使用FileUtils一次性读取文件内容
            String content = FileUtils.readFileToString(new File(filePath), "UTF-8");
            System.out.println(content);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

3. 大文件读取

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class ReadFileWithBufferedReader {
    public static void main(String[] args) {
        String filePath = "example.txt";
        try (BufferedReader reader = new BufferedReader(new FileReader(filePath))) {
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

