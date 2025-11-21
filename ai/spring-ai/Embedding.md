向量

将一个字、一个词转化成多维向量，就可以比较这个词是否相近，相近就可能是需要的结果

一、简单模拟将文本转换成向量，

```java
public static void main(String[] args) {
        OpenAiEmbeddingModel embeddingModel = OpenAiEmbeddingModel.builder()
                .baseUrl("http://langchain4j.dev/demo/openai/v1")
                .apiKey("demo")
                .modelName("text-embedding-3-small")
                .build();
        
        Response<Embedding> embed = embeddingModel.embed("你好，我叫裴立志");
        System.out.println(embed.content().toString());
       //  向量的维度 ： text-embedding-3-small模型默认是1536
        System.out.println(embed.content().vector().length);
    }
```

不同的模型转换的向量维度不同

二、向量数据库

将向量持久化到数据库中，用于存储向量。

1. 首先下载redis

   ```
   docker run -d  -p 6379:6379 redis/redis-stack-server:latest
   ```

2. 配置xml

   ```xml
   <dependency>
   	<groupId>dev.langchain4j</groupId>
   	<artifactId>langchain4j-redis</artifactId>
   	<version>${langchain4j.version}</version>
   </dependency>
   ```

3. java 指令

   ```java
   OpenAiEmbeddingModel embeddingModel = OpenAiEmbeddingModel.builder()
                   .baseUrl("http://langchain4j.dev/demo/openai/v1")
                   .apiKey("demo")
                   .modelName("text-embedding-3-small")
                   .build();
   
           RedisEmbeddingStore embeddingStore = RedisEmbeddingStore.builder()
                   .host("123.57.235.173")
                   .port(6379)
                   // 设置向量维度
                   .dimension(1536)
                   .build();
   
           // 生成向量
           Response<Embedding> embed = embeddingModel.embed("我是裴立志");
   
           // 存储向量
           embeddingStore.add(embed.content());
   ```

   四、检查向量相似度

   ```java
   OpenAiEmbeddingModel embeddingModel = OpenAiEmbeddingModel.builder()
                   .baseUrl("http://langchain4j.dev/demo/openai/v1")
                   .apiKey("demo")
                   .modelName("text-embedding-3-small")
                   .build();
   
           RedisEmbeddingStore embeddingStore = RedisEmbeddingStore.builder()
                   .host("123.57.235.173")
                   .port(6379)
                   // 设置向量维度
                   .dimension(1536)
                   .build();
           // 生成向量
           Response<Embedding> embed = embeddingModel.embed("我的名字是裴立志");
           
           List<EmbeddingMatch<TextSegment>> result = embeddingStore.findRelevant(embed.content(), 4);
           for (EmbeddingMatch<TextSegment> embeddingMatch : result) {
               String string = embeddingMatch.toString();
             // 会计算出一个得分来，得分越高越相似
               System.out.println( embeddingMatch.score());
           }
   ```

   五、模糊搜索数据

   ```java
   OpenAiEmbeddingModel embeddingModel = OpenAiEmbeddingModel.builder()
                   .baseUrl("http://langchain4j.dev/demo/openai/v1")
                   .apiKey("demo")
                   .modelName("text-embedding-3-small")
                   .build();
   
           RedisEmbeddingStore embeddingStore = RedisEmbeddingStore.builder()
                   .host("123.57.235.173")
                   .port(6379)
                   // 设置向量维度
                   .dimension(1536)
                   .build();
           // 生成向量
           TextSegment textSegment1 = TextSegment.textSegment("客服电话是400-8558558");
           TextSegment textSegment2 = TextSegment.textSegment("客服工作时间是周一到周五");
           TextSegment textSegment3 = TextSegment.textSegment("客服投诉电话是400-8668668");
           Response<Embedding> embed1 = embeddingModel.embed(textSegment1);
           Response<Embedding> embed2 = embeddingModel.embed(textSegment2);
           Response<Embedding> embed3 = embeddingModel.embed(textSegment3);
   
           //// 存储向量
           //embeddingStore.add(embed1.content(), textSegment1);
           //embeddingStore.add(embed2.content(), textSegment2);
           //embeddingStore.add(embed3.content(), textSegment3);
   
           // 生成向量
           Response<Embedding> embed = embeddingModel.embed("客服电话多少");
   
           // 查询
           List<EmbeddingMatch<TextSegment>> result = embeddingStore.findRelevant(embed.content(), 5, 0.8D);
           for (EmbeddingMatch<TextSegment> embeddingMatch : result) {
               System.out.println(embeddingMatch.embedded().text() + ",分数为：" + embeddingMatch.score());
           }
   ```

   





