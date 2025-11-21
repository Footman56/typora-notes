AiService 的作用

+ 能够生成特定接口的代理对象，从而使得在调用代理对象方法时，能间接的调用大模型。
+ 使用tool 工具能够方便与实际业务像关联



一、基础写法

Write 是一个接口

```java
public interface Writer {
    String write(String prompt);
}
```

```java

public static void main(String[] args) {
        ChatLanguageModel model = OpenAiChatModel.builder()
                .apiKey("sk-svcacct-8wM70txGF2cYVj4v1pmKS8obFj0QQB2UVGeHblTvruMfkEcgOAn0afhgYd9P6EdUT3BlbkFJrsb_Y7MsbAr0HlH1pm6Dtjv07tK_D2KK7J73cDVn_U2nD9kldVyZeddoCnVJ8HsA")
                .build();

        Writer writer = AiServices.create(Writer.class, model);
        String write = writer.write("你是一个作家，请写一篇关于母亲的文章，内容不超过100字");
        System.out.println(write);
    }
```

二、 注解方式

```java
public interface Writer {

    @SystemMessage("你是一个作家,请根据输入的题材写一篇文章,文章字数不少于{{num}}")
    @Moderate
    String write(@UserMessage String prompt, @V("num") Integer num);
}
```

```java
ChatLanguageModel chatModel = OpenAiChatModel.builder()
                .apiKey("sk-svcacct-8wM70txGF2cYVj4v1pmKS8obFj0QQB2UVGeHblTvruMfkEcgOAn0afhgYd9P6EdUT3BlbkFJrsb_Y7MsbAr0HlH1pm6Dtjv07tK_D2KK7J73cDVn_U2nD9kldVyZeddoCnVJ8HsA")
                .build();


        ModerationModel moderationModel = OpenAiModerationModel.builder()
                .apiKey("sk-svcacct-8wM70txGF2cYVj4v1pmKS8obFj0QQB2UVGeHblTvruMfkEcgOAn0afhgYd9P6EdUT3BlbkFJrsb_Y7MsbAr0HlH1pm6Dtjv07tK_D2KK7J73cDVn_U2nD9kldVyZeddoCnVJ8HsA")
                .build();

        Writer writer = AiServices.builder(Writer.class)
                .chatLanguageModel(chatModel)
                .moderationModel(moderationModel)
                .build();
        String write = writer.write("请写一篇关于母亲的文章", 100);
        System.out.println(write);
```

@SystemMessage 表示系统参数，用于配置系统信息，用于声明AI角色

@UserMessage 用于表示用户数据，

@V 自定义参数

@Moderate 用于声明校验敏感词模型

三、 注册成Bean

```java
@Configuration
public class AiBeanConfiguration {

    @Bean
    public Writer writer() {
        ChatLanguageModel chatModel = OpenAiChatModel.builder()
                .apiKey("sk-svcacct-8wM70txGF2cYVj4v1pmKS8obFj0QQB2UVGeHblTvruMfkEcgOAn0afhgYd9P6EdUT3BlbkFJrsb_Y7MsbAr0HlH1pm6Dtjv07tK_D2KK7J73cDVn_U2nD9kldVyZeddoCnVJ8HsA")
                .build();
        
        ModerationModel moderationModel = OpenAiModerationModel.builder()
                .apiKey("sk-svcacct-8wM70txGF2cYVj4v1pmKS8obFj0QQB2UVGeHblTvruMfkEcgOAn0afhgYd9P6EdUT3BlbkFJrsb_Y7MsbAr0HlH1pm6Dtjv07tK_D2KK7J73cDVn_U2nD9kldVyZeddoCnVJ8HsA")
                .build();

        Writer writer = AiServices.builder(Writer.class)
                .chatLanguageModel(chatModel)
                .moderationModel(moderationModel)
                .build();
        return writer;
    }
}
```

```java
@Resource
    private Writer writer;

    @GetMapping("/write")
    public String write(String query) {
        String write = writer.write(query, 100);
        return write;
    }
```



四、配置记忆模型 ChatMemory

一般来说model 调用是无记忆的，不会记录上次输入的内容，想要实现有记忆的话，一种笨方法就是在请求的时候手动将之前所有的对话过程（用户输入、AI 输出）传递给model 。第二种就是使用记忆模型，用于根据用户去记录输入。（实现原理就是每次请求的时候讲本次输入缓存起来）

ChatMemory 有两个实现

+ MessageWindowChatMemory ： 窗口记忆，设置最大记录信息，超过的话就将最初的信息删除掉
+ TokenWindowChatMemory【推荐】token 记忆，设置最大token长度，超过长度的话就将最初的信息删除

ChatMemoryStore: 用于持久化消息，默认实现是InMemoryChatMemoryStore ，用于存入到内存中

```java
 @Bean
    public NamingMaster namingMaster() {
        ChatLanguageModel chatModel = OpenAiChatModel.builder()
                .apiKey("sk-svcacct-8wM70txGF2cYVj4v1pmKS8obFj0QQB2UVGeHblTvruMfkEcgOAn0afhgYd9P6EdUT3BlbkFJrsb_Y7MsbAr0HlH1pm6Dtjv07tK_D2KK7J73cDVn_U2nD9kldVyZeddoCnVJ8HsA")
                .build();

        TokenWindowChatMemory tokenWindowChatMemory = TokenWindowChatMemory.builder()
                .maxTokens(1000, new OpenAiTokenizer(OpenAiChatModelName.GPT_3_5_TURBO.toString()))
                .build();

        return AiServices.builder(NamingMaster.class)
                .chatLanguageModel(chatModel)
                .chatMemory(tokenWindowChatMemory)
                .build();
    }
```

五、用户独立

在提供成controller接口的时候，不同的用户请求时都是用相同的chatmemory的话，可能导致回复混乱。解决方式时不同用户使用不能id来区别记忆

注：在chatMemoryProvider 设置 Chatmemory时需要不同的UserId 返回不同的记忆体

```java
@Bean
    public NamingMaster namingMaster() {
        ChatLanguageModel chatModel = OpenAiChatModel.builder()
                .apiKey("sk-svcacct-8wM70txGF2cYVj4v1pmKS8obFj0QQB2UVGeHblTvruMfkEcgOAn0afhgYd9P6EdUT3BlbkFJrsb_Y7MsbAr0HlH1pm6Dtjv07tK_D2KK7J73cDVn_U2nD9kldVyZeddoCnVJ8HsA")
                .build();


        return AiServices.builder(NamingMaster.class)
                .chatLanguageModel(chatModel)
                .chatMemoryProvider(userId -> TokenWindowChatMemory
                        .builder()
                        .maxTokens(100, new OpenAiTokenizer(OpenAiChatModelName.GPT_3_5_TURBO.toString()))
                        .build())
                .build();
    }
```

```java
public interface NamingMaster {


    @SystemMessage("你是一个熟悉中国历史，有丰富的文学底蕴，请你根据描述生成一个名称，需要保证名称比较独特，避免重名，并且名称需要能从诗词歌赋中找到出处")
    String naming(@MemoryId String memoryId, @UserMessage String desc);
}
```

六、tools 

在一些情景中AI并不是万能的，有的时候需要程序猿去编写对应的与实际业务相符的方法，AI就能调用对应的方法来获取结果，并且将运行的结果作为模型的输入，最终输出符合用户的结论

正常来说，AI 是不知道实时信息的

```java
 ChatLanguageModel chatModel = OpenAiChatModel.builder()
                    .apiKey("sk-svcacct-8wM70txGF2cYVj4v1pmKS8obFj0QQB2UVGeHblTvruMfkEcgOAn0afhgYd9P6EdUT3BlbkFJrsb_Y7MsbAr0HlH1pm6Dtjv07tK_D2KK7J73cDVn_U2nD9kldVyZeddoCnVJ8HsA")
                    .build();

        String resut = chatModel.generate("今天是几月几号");
        System.out.println("resut = " + resut);
```

显示的结果是乱的

<img src="https://raw.githubusercontent.com/Footman56/images/master/img202502071139614.png" alt="image-20250201100752835" style="zoom:50%;" />

需要使用Tool 来获取具体的时间

```java
ChatLanguageModel chatModel = OpenAiChatModel.builder()
                    .apiKey("sk-svcacct-8wM70txGF2cYVj4v1pmKS8obFj0QQB2UVGeHblTvruMfkEcgOAn0afhgYd9P6EdUT3BlbkFJrsb_Y7MsbAr0HlH1pm6Dtjv07tK_D2KK7J73cDVn_U2nD9kldVyZeddoCnVJ8HsA")
                    .build();

        // 创建工具方法
        ToolSpecification toolSpecification = ToolSpecifications.toolSpecificationFrom(HuochaiTools.class.getMethod("getDate"));

        ChatMessage userMessage = UserMessage.from("今天是几月几号？");

        List<ChatMessage> chatMessages = new ArrayList<>();
        chatMessages.add(userMessage);

        Response<AiMessage> generate = chatModel.generate(chatMessages, toolSpecification);
        // 注此时获取的是generate.content().text() 为null,是因为有工具请求
        if (generate.content().hasToolExecutionRequests()){
            chatMessages.add(generate.content());
            for (ToolExecutionRequest toolExecutionRequest : generate.content().toolExecutionRequests()) {
                // 获取请求的需要接口
                String method = toolExecutionRequest.name();
                try {
                    Method method1 = HuochaiTools.class.getMethod(method);
                    String result = (String)method1.invoke(new HuochaiTools());
                    System.out.println("result = " + result);

                    // 封装成ToolExecutionResultMessage 继续请求AI
                    ToolExecutionResultMessage toolExecutionResultMessage = ToolExecutionResultMessage.from(toolExecutionRequest.id(), toolExecutionRequest.name(), result);
                    chatMessages.add(toolExecutionResultMessage);
                    Response<AiMessage> toolResponse = chatModel.generate(chatMessages);
                    System.out.println("toolResponse = " + toolResponse.content().text());
                } catch (Exception e) {
                    throw new RuntimeException(e);
                }
            }
        }
```

使用AIServices  封装的形式

```java
public class HuochaiTools {
    
    @Tool(name = "获取日期", value =  "用于获取当前日期")
    public String getDate() {
        LocalDateTime localDateTime = LocalDateTime.now();
        DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        return localDateTime.format(dateTimeFormatter);
    }
    
    @Tool("用于根据指定日期获取注册的用户")
    public List<User> getUserInfoByDate(String date) {
        System.out.println("入参 date = " + date);
        User user1 = new User("张三", 18, "男");
        User user2 = new User("李四", 19, "女");
        User user3 = new User("王五", 20, "男");
        User user4 = new User("赵六", 21, "女");
        User user5 = new User("孙七", 22, "男");
        return List.of(user1, user2, user3, user4, user5);
    }
}
```

```java
public interface AIToolsTest {

    @SystemMessage("今天是几月几号")
    String getDate();

    @SystemMessage("先获取当前日期，在根据当前日期查询注册的用户，根据这些用户去满足客户需求")
    String getUserInfo(String date);
}
```

```java
public static void main(String[] args) {

        ChatLanguageModel chatModel = OpenAiChatModel.builder()
                .apiKey("sk-svcacct-8wM70txGF2cYVj4v1pmKS8obFj0QQB2UVGeHblTvruMfkEcgOAn0afhgYd9P6EdUT3BlbkFJrsb_Y7MsbAr0HlH1pm6Dtjv07tK_D2KK7J73cDVn_U2nD9kldVyZeddoCnVJ8HsA")
                .build();

        TokenWindowChatMemory tokenWindowChatMemory = TokenWindowChatMemory.builder()
                .maxTokens(1000, new OpenAiTokenizer(OpenAiChatModelName.GPT_3_5_TURBO.toString()))
                .build();

        AIToolsTest toolsTest = AiServices.builder(AIToolsTest.class)
                .chatLanguageModel(chatModel)
                .chatMemory(tokenWindowChatMemory)
                .tools(new HuochaiTools())
                .build();


        String userInfo = toolsTest.getUserInfo("根据用户数据去总结出这些用户中年龄分布，性别比例分布");
        System.out.println("userInfo = " + userInfo);
```

难点在于让AI去使用工具来查询数据，解决方式是设置成系统提示词来规定实现的步骤，系统提示词中描述尽量与工具描述保持一致才能更好地使用工具。用户提示词来说明结果的处理，系统提示词可以描述成怎么获取结果

为了让LLM使用正确的参数。需要提供明确的信息

+ 工具的名称
+ 描述改工作的作用以及何时使用
+ 每个工具参数的描述

