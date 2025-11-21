使用LangChain4j

LangChan4j 可以等同于多个大模型调用联，封装了一些大模型的调用方法，在使用的时候可以综合多个模型的优势来实现项目功能

比如可以通过openAi 来有数promt, 有了优化后的promt可以调用别的模型来生成数据

```xml
<langchain4j.version>0.27.1</langchain4j.version>

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
            <groupId>org.tinylog</groupId>
            <artifactId>tinylog-impl</artifactId>
            <version>2.6.2</version>
        </dependency>
        <dependency>
            <groupId>org.tinylog</groupId>
            <artifactId>slf4j-tinylog</artifactId>
            <version>2.6.2</version>
        </dependency>
```



# 与OpenAi对话

注：apices 设置为demo  的话，可以免费调用openAi模型

`下面方式使用的时候，是单次对话，后面再次对话的时候是不会记录之前的结果的`

```java
    public static void main(String[] args) {
        ChatLanguageModel model = OpenAiChatModel
                .builder()
                .apiKey("demo")
                .modelName("gpt-4o-mini")
                .build();
        String answer = model.generate("你好，我是裴立志？");
        System.out.println(answer);
    }
```

下面实现会记录之前的结果

```java
 public static void main(String[] args) {
        ChatLanguageModel model = OpenAiChatModel.builder()
                .apiKey("demo")
                .modelName("gpt-4o-mini")
                .build();
        UserMessage userMessage1 = UserMessage.userMessage("你好，我是裴立志");
        Response<AiMessage> response1 = model.generate(userMessage1);
        AiMessage aiMessage1 = response1.content(); // 大模型的第一次响应
        System.out.println(aiMessage1.text());
        System.out.println("----");
        List<ChatMessage> chatMessages = new ArrayList<>();
        chatMessages.add(userMessage1);
        chatMessages.add(aiMessage1);

        // 下面一行代码是重点
        UserMessage userMessage2 = UserMessage.userMessage("我叫什么");
        chatMessages.add(userMessage2);
        Response<AiMessage> response2 = model.generate(chatMessages);
        AiMessage aiMessage2 = response2.content(); // 大模型的第二次响应
        System.out.println(aiMessage2.text());
        chatMessages.add(aiMessage2);

        System.out.println("----");
        UserMessage userMessage3 = UserMessage.userMessage("我今年13岁啦");
        chatMessages.add(userMessage3);
        Response<AiMessage> response3 = model.generate(chatMessages);
        AiMessage aiMessage3 = response3.content();
        System.out.println(aiMessage3.text());
        chatMessages.add(aiMessage3);

        System.out.println("----");
        UserMessage userMessage4 = UserMessage.userMessage("我叫什么?");
        chatMessages.add(userMessage4);
        Response<AiMessage> response4 = model.generate(chatMessages);
        AiMessage aiMessage4 = response4.content();
        System.out.println(aiMessage4.text());
        chatMessages.add(aiMessage4);

        System.out.println("----");
        UserMessage userMessage5 = UserMessage.userMessage("我今年几岁?");
        chatMessages.add(userMessage5);
        Response<AiMessage> response5 = model.generate(chatMessages);
        AiMessage aiMessage5 = response5.content();
        System.out.println(aiMessage5.text());
        chatMessages.add(aiMessage5);
    }
```

# 与智普模型来对话



# 打印机式输出

```java
 StreamingChatLanguageModel model = OpenAiStreamingChatModel.builder()
                .apiKey("sk-svcacct-8wM70txGF2cYVj4v1pmKS8obFj0QQB2UVGeHblTvruMfkEcgOAn0afhgYd9P6EdUT3BlbkFJrsb_Y7MsbAr0HlH1pm6Dtjv07tK_D2KK7J73cDVn_U2nD9kldVyZeddoCnVJ8HsA")
                .build();
        model.generate("你好，你是谁？", new StreamingResponseHandler<AiMessage>() {
            @Override
            public void onNext(String token) {

                System.out.print(token);

                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            @Override
            public void onError(Throwable error) {
                System.out.println(error);
            }
        });
```





# 检查输入是否有敏感词

```java
ModerationModel moderationModel = OpenAiModerationModel.withApiKey("demo");
        Response<Moderation> response = moderationModel.moderate("我杀人了");
        System.out.println(response.content().flaggedText());
```

如果返回的内容为null 就说明不是敏感词，如果返回的结果有内容，内容就是敏感词



# 整合springboot 

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
	<groupId>dev.langchain4j</groupId>
	<artifactId>langchain4j-open-ai-spring-boot-starter</artifactId>
	<version>0.27.1</version>
</dependency>
```

配置api-key

```
langchain4j.open-ai.chat-model.api-key=sk-svcacct-8wM70txGF2cYVj4v1pmKS8obFj0QQB2UVGeHblTvruMfkEcgOAn0afhgYd9P6EdUT3BlbkFJrsb_Y7MsbAr0HlH1pm6Dtjv07tK_D2KK7J73cDVn_U2nD9kldVyZeddoCnVJ8HsA
```

接口使用

ChatLanguageModel 接口实现取决于引入什么pom文件，引入open ai 的jar，实现就是openai的

```java
@Autowired
    private ChatLanguageModel chatLanguageModel;

    @GetMapping("/hello")
    public String hello(){
        return chatLanguageModel.generate("你好啊");
    }
```



绘画

```java
 ImageModel model = OpenAiImageModel.builder()
                .apiKey("sk-svcacct-pBhWc2O5qP8Hu60TS8c4cCCUeBvu7Oi9kE8PoUohNtBQ_eZGSqkrh0yiai4F1FET3BlbkFJy9X-9ug4z3BuSm0o6TqBEM90g0X11d8ElTj4PNEw1vS6nBZiRbdNq3WppsUI0AA")
                .modelName(DALL_E_3)
                .build();

        Response<Image> response = model.generate("海边的落日");

        URI url = response.content().url();
        System.out.println(url); // Donald Duck is here :)
```

