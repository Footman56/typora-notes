# 一、引入xml

```xml
<langchain4j.version>1.0.0-alpha1</langchain4j.version>

<dependency>
  <groupId>dev.langchain4j</groupId>
  <artifactId>langchain4j</artifactId>
  <version>${langchain4j.version}</version>
</dependency>
<dependency>
  <groupId>dev.langchain4j</groupId>
  <artifactId>langchain4j-open-ai</artifactId>
  <version>${langchain4j.version}</version>
</dependency>

<dependency>
  <groupId>dev.langchain4j</groupId>
  <artifactId>langchain4j-open-ai-spring-boot-starter</artifactId>
  <version>${langchain4j.version}</version>
</dependency>

<dependency>
  <groupId>dev.langchain4j</groupId>
  <artifactId>langchain4j-redis</artifactId>
  <version>${langchain4j.version}</version>
</dependency>

<dependency>
  <groupId>dev.langchain4j</groupId>
  <artifactId>langchain4j-easy-rag</artifactId>
  <version>${langchain4j.version}</version>
</dependency>
```

踩坑：之前langchain4j.version 版本是0.27.1 时引入langchain4j-easy-rag 时是需要BlankDocumentException ，但是版本太低没有，需要升级版本到1.0.0-alpha1。

# 二、配置文件

```java
/**
     * openai模型
     * @return
     */
    @Bean
    @Primary
    public ChatLanguageModel openaiChatModel() {
        ChatLanguageModel chatModel = OpenAiChatModel.builder()
                .apiKey("sk-svcacct-8wM70txGF2cYVj4v1pmKS8obFj0QQB2UVGeHblTvruMfkEcgOAn0afhgYd9P6EdUT3BlbkFJrsb_Y7MsbAr0HlH1pm6Dtjv07tK_D2KK7J73cDVn_U2nD9kldVyZeddoCnVJ8HsA")
                .build();
        return chatModel;
    }

    @Bean
    public OpenAiTokenizer tokenizer() {
        OpenAiTokenizer openAiTokenizer = new OpenAiTokenizer(OpenAiChatModelName.GPT_3_5_TURBO.toString());
        return openAiTokenizer;
    }

    /**
     * token记忆模型，注意设置maxTokens 的大小，不能太小，太小的话容易被过滤掉，导致没有message返回报错
     * @return
     */
    @Bean
    public TokenWindowChatMemory tokenWindowChatMemory() {
        return TokenWindowChatMemory.builder()
                .maxTokens(500, tokenizer())
                .build();
    }

    /**
     * redis 来作为向量数据库
     * @return
     */
    @Bean
    public RedisEmbeddingStore embeddingStore() {
        RedisEmbeddingStore embeddingStore = RedisEmbeddingStore.builder()
                .host("123.57.235.173")
                .port(6379)
                // 设置向量维度
                .dimension(1536)
                .build();
        return embeddingStore;
    }


    /**
     * 嵌入模型，用于将文本转换成向量
     * @return
     */
    @Bean
    public OpenAiEmbeddingModel openAiEmbeddingModel() {
        OpenAiEmbeddingModel embeddingModel = OpenAiEmbeddingModel.builder()
                .baseUrl("http://langchain4j.dev/demo/openai/v1")
                .apiKey("demo")
                .modelName("text-embedding-3-small")
                .build();
        return embeddingModel;
    }
    /**
     * 嵌入导管
     * 将文档嵌入到向量数据库中
     * @return
     */
    @Bean
    public EmbeddingStoreIngestor embeddingStoreIngestor() {
        return EmbeddingStoreIngestor.builder()
                // 为每个 Document 添加 userId 元数据条目，便于后续过滤;在嵌入之前对文档进行清理、增强或格式化时非常有用。
                .documentTransformer(document -> {
                    document.metadata().put("userId", "12345");
                    return document;
                })
                // 将每个 Document 拆分为 1000 个 token 的 TextSegment，具有 200 个 token 的重叠;文档较大且您希望将其拆分为较小的 TextSegment 时非常有用，以提高相似度搜索的质量并减少发送给 LLM 的提示词的大小和成本
                .documentSplitter(DocumentSplitters.recursive(1000, 200, tokenizer()))

                // 为每个 TextSegment 添加 Document 的名称，以提高搜索质量;您希望在嵌入之前对 TextSegment 进行清理、增强或格式化时非常有用。
                .textSegmentTransformer(textSegment -> TextSegment.from(
                        textSegment.metadata().getString("file_name") + "\n" + textSegment.text(),
                        textSegment.metadata() 
                ))
                .embeddingStore(embeddingStore())
                .embeddingModel(openAiEmbeddingModel())
                .build();
    }
```

EmbeddingStoreIngestor 提供了一个标准化处理文件到向量库的流程，有什么需要执行在documentTransformer、documentSplitter、textSegmentTransformer 做拓展即可。

**DocumentTransformer** ： 以文档的角度进行处理，最后的结果也是文档Document 一般来说在这一步可以对Metadata进行拓展

```java
public static final String FILE_NAME = "file_name";
public static final String ABSOLUTE_DIRECTORY_PATH = "absolute_directory_path";
public static final String URL = "url";
private final String text;
private final Metadata metadata;
```

**DocumentSplitter**： 描述的是怎么将Document 转化成List<TextSegment> 不同的实现有不同的切分方法，没有标准答案，适合就是好的

**TextSegmentTransformer** 作用是将List<TextSegment> 转化成List<TextSegment>，对每一个TextSegment 执行transform 方法并且过滤非null的数据，对每个TextSegment 进行格式化处理。

# 三、索引文件

分成两步：第一个读取文件、第二步存储向量数据库

这种处理比较粗糙，是将整个文件作为向量存储的，应该有比较细致化的操作。比较根据\n来区别的。

```java
    @Resource
    private EmbeddingStoreIngestor embeddingStoreIngestor; 

/**
     * 获取目录下所有的文件
     * @param path
     * @return
     */
    public List<Document> findTextDocuments(String path) {
        return FileSystemDocumentLoader.loadDocuments(path, new ApacheTikaDocumentParser());
    }

```

读取文件转化成Document 对象，需要结合同的文件类型来处理。

- `TextDocumentParser`来自`langchain4j`模块，可以解析纯文本格式的文件（例如TXT，HTML，MD等）

- `ApachePdfBoxDocumentParser`来自`langchain4j-document-parser-apache-pdfbox`可以解析 PDF 文件的模块

- `ApachePoiDocumentParser`来自`langchain4j-document-parser-apache-poi`模块，它可以解析 MS Office 文件格式（例如 DOC、DOCX、PPT、PPTX、XLS、XLSX 等）

- `ApacheTikaDocumentParser`该`langchain4j-document-parser-apache-tika`模块可以自动检测和解析几乎所有现有的文件格式

  暂时未知怎么处理markdown文件

## markdown文件

1. 引入pom

```xml
  <dependency>
            <groupId>dev.langchain4j</groupId>
            <artifactId>langchain4j-document-parser-apache-tika</artifactId>
            <version>${langchain4j.version}</version>
        </dependency>

        <!-- Apache Tika -->
        <dependency>
            <groupId>org.apache.tika</groupId>
            <artifactId>tika-core</artifactId>
            <version>2.5.0</version>
        </dependency>
```

2. 配置类

```java
   /**
     * 嵌入模型，用于将文本转换成向量
     * @return
     */
    @Bean
    public OpenAiEmbeddingModel openAiEmbeddingModel() {
        OpenAiEmbeddingModel embeddingModel = OpenAiEmbeddingModel.builder()
                .apiKey("sk-svcacct-uwWGFr8eyld1PNDZpLyDsHV83XQ5CsuSjC2yP0FjWVPHRyGS-ShAifRkMnY1NixanT3BlbkFJr-mgdbuAbmbvhYp0zcdqbew2gybdbjfEqyjvsuiPebrH8Cjye-E6I_BD9ojRkEVAA")
                .modelName("text-embedding-3-small")
                .build();
        return embeddingModel;
    }
```

踩坑：

```
dev.ai4j.openai4j.OpenAiHttpException: Maximum number of tokens per request for demonstration purposes is 10000. If you wish to use more, please use your own OpenAI API key.
```

解决方式：不要使用 apiKey = demo 、 baseUrl = http://langchain4j.dev/demo/openai/v1。 使用自己实在的key

```java
public List<Document> findMarkdownDocuments(String path) {
        File markdownPath = new File(path);
        if (!markdownPath.exists()) {
            return null;
        }
        List<Document> documents = new ArrayList<>();
        try {
            if (markdownPath.isDirectory()) {
                File[] files = markdownPath.listFiles();
                for (File file : files) {
                    if (file.isFile() && file.getName().endsWith(".md")) {
                        Tika tika = new Tika();
                        // 使用 Tika 解析文件，提取文本内容
                        String content = tika.parseToString(file);
                        Document document = new Document(content);
                        documents.add(document);
                    }
                }
            } else {
                Tika tika = new Tika();
                // 使用 Tika 解析文件，提取文本内容
                String content = tika.parseToString(markdownPath);
                Document document = new Document(content);
                documents.add(document);
            }
        } catch (IOException | TikaException e) {
            log.error("解析文件失败", e);
        }
        return documents;
    }
```

```java
    @GetMapping("/embedding-md")
    public void embeddingMarkDown() {
        List<Document> textDocuments = documentService.findMarkdownDocuments("/Users/peilizhi/typora/linux");
        documentService.inject(textDocuments);
    }
```



```java
/**
     * 存入向量数据库
     * @param documents
     */
    public void inject(List<Document> documents) {
        if (CollectionUtils.isEmpty(documents)) {
            return;
        }
      // 在底层嵌入的时候，先
        embeddingStoreIngestor.ingest(documents);
    }
```

**embeddingStoreIngestor.ingest()**

1. 如果配置documentTransformer 的话，就执行transformAll() 
2. 如果配置documentSplitter 的话就执行splitAll() 方法，转化成List<TextSegment> segments；否则采用默认toTextSegment()实现
3. 如果配置textSegmentTransformer 的话就执行transformAll() 
4. embeddingModel 进行向量化
5. 执行embeddingStore.addAll() [以RedisEmbeddingStore 为例]
   1. 根据向量的个数去生成唯一的标识符
   2. 以key-value的形式存入redis 中 
      + key:  前缀 +  唯一标识
      + value json形式的数据，有vector： 向量、text:实际文本内容、metadata

`metadata 提供了灵活拓展的功能，能够赋予多余的属性，有了这些属性就可以做很多贴合业务的功能。`

`向量与文本采用下标序号对应的方式`

# 四、手动操作

一般来说都是在后台处理的，可以通过定时任务或者手动上传文件目录来设置的。

```java
 @Resource
    private DocumentService documentService;

    @GetMapping("/embedding")
    public void  embedding(){
        List<Document> textDocuments = documentService.findTextDocuments("/Users/peilizhi/Desktop/txt-dir");
        documentService.inject(textDocuments);
    }
```

踩坑：

```java
a.d.huggingface.tokenizers.jni.LibUtils  : library not found in classpath: native/lib/osx-x86_64/cpu/libtokenizers.dylib
2025-02-06T15:48:22.107+08:00 ERROR 53190 --- [nio-8080-exec-2] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Handler dispatch failed: java.lang.ExceptionInInitializerError] with root cause

ai.djl.engine.EngineException: Unexpected flavor: cpu
```

解决方式：

```xml
  <!-- 示例：升级到最新版本 -->
        <dependency>
            <groupId>ai.djl.huggingface</groupId>
            <artifactId>tokenizers</artifactId>
            <version>0.28.0</version> <!-- 使用最新版本 -->
        </dependency>
```

踩坑：

```
Error parsing vector similarity query: query vector blob size (1536) does not match index's expected size (6144).
```

主要是查询问题时与构建向量数据库时使用不同的嵌入模型，

```java
/**
     * 获取结果的数据
     * 必须采用相同的嵌入模型、向量维度和向量数据库
     * @return
     */
    @Bean
    public EmbeddingStoreContentRetriever embeddingStoreContentRetriever() {
        return EmbeddingStoreContentRetriever.builder()
                .embeddingStore(embeddingStore())
                .embeddingModel(openAiEmbeddingModel())
                .maxResults(2)
                .minScore(0.8)
                .build();
    }
```

```java
  @GetMapping("/embedding-query")
    public String embeddingQuery(String query) {
        RagService ragService = AiServices.builder(RagService.class)
                .chatLanguageModel(chatLanguageModel)
                .chatMemory(tokenWindowChatMemory)
          // 使用同样的嵌入模型来处理问题
                .contentRetriever(embeddingStoreContentRetriever)
                .build();
        String chart = ragService.chart(query);
        return chart;
    }
```

踩坑：

```
2025-02-06T16:34:44.023+08:00 ERROR 81772 --- [nio-8080-exec-2] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed: java.lang.IllegalArgumentException: messages cannot be null or empty] with root cause
java.lang.IllegalArgumentException: messages cannot be null or empty
```

```
# 有可能时token设置的太小，导致别移除，所以没有结果，改成MessageWindowChatMemory这个模型就行啦，或者适当调大token
```

