# Resource

Resource 是一个统一的概念，所有File、URL、文件资源等都可以用Resoure表示

`Resource` 提供了 `getInputStream()` 方法，允许直接读取资源内容，而无需关心资源的实际来源

想要获得资源的话就可以 通过读取inputStream 来实现

**如何选择合适的 `Resource` 实现？**
   - `ClassPathResource`: 用于访问类路径下的资源。
   - `FileSystemResource`: 用于访问文件系统中的资源。
   - `UrlResource`: 用于基于URL的资源，如HTTP或FTP资源。
   - `ServletContextResource`: 专为 Web 应用程序设计，用于访问`ServletContext`中的资源。

```java
  byte[] data = "hello world".getBytes();
        Resource resource = new ByteArrayResource(data);
        try (InputStream is = resource.getInputStream()) {
            // 读取和处理资源内容
            byte[] bytes = new byte[1024];
            int bytesRead = is.read(bytes);
            while (bytesRead != -1) {
                // 处理数据
                System.out.print(new String(bytes, 0, bytesRead));
                bytesRead = is.read(bytes);
            }
        }
```



# ResourceLoader

 用于加载Resource

```java
 DefaultResourceLoader loader = new DefaultResourceLoader();
// 从类路径加载资源
        Resource classpathResource = loader.getResource("classpath:application.properties");
        System.out.println("Classpath Exists= " + classpathResource.exists());

        // 加载文件系统中的资源
        Resource fileResource = loader.getResource("file:/Users/peilizhi/javaProjects/spring-reading/spring-resources/spring-resource-resourceLoader/myfile1.txt");
        System.out.println("File Exists = " + fileResource.exists());
```



# DocumentLoader

用于加载xml文件

~~~java
```java
/**
 * 用于加载 XML Document 的策略接口。
 *
 * @author Rob Harrop
 * @since 2.0
 * @see DefaultDocumentLoader
 */
public interface DocumentLoader {
    /**
     * 从提供的 InputSource source 加载一个 Document document。
     * @param inputSource 要加载的文档的来源
     * @param entityResolver 用于解析任何实体的解析器
     * @param errorHandler 用于在加载文档过程中报告任何错误
     * @param validationMode 验证的类型 
     * （org.springframework.util.xml.XmlValidationModeDetector#VALIDATION_DTD DTD
     * 或 org.springframework.util.xml.XmlValidationModeDetector#VALIDATION_XSD XSD)
     * @param namespaceAware 如果需要提供对XML命名空间的支持，则为 true
     * @return 加载的 Document document
     * @throws Exception 如果发生错误
     */
    Document loadDocument(
        InputSource inputSource, EntityResolver entityResolver,
        ErrorHandler errorHandler, int validationMode, boolean namespaceAware)
        throws Exception;

}
~~~

需要从Resource中获取InputStream

```java
 try {
            DefaultResourceLoader loader = new DefaultResourceLoader();
            // 创建要加载的资源对象
            Resource resource = loader.getResource("classpath:sample.xml");

            // 创建 DocumentLoader 实例
            DefaultDocumentLoader documentLoader = new DefaultDocumentLoader();

            // 加载和解析 XML 文档
            Document document = documentLoader.loadDocument(new InputSource(resource.getInputStream()), null, null, 0, true);

            // 打印 XML 文档的内容
            printDetailedDocumentInfo(document);
        } catch (Exception e) {
            e.printStackTrace();
        }
```

# ResourcePatternResolver

用于加载多个Resource

```java
 ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();

        // 加载所有匹配的类路径资源
        Resource[] resources = resolver.getResources("classpath*:*.properties");
        for (Resource resource : resources) {
            System.out.println("Classpath = " + resource.getFilename());
        }
```

