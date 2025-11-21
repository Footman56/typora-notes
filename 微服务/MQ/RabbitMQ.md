





# 零、AMQP

<img src="/Users/mac/Pictures/截图/屏幕快照 2021-04-19 21.43.01.png" style="zoom:40%;" />



```
什么是MQ（Message Queue）消息队列： 通过典型的生产者和消费者模型，生产者不断地向队列产生消息，消费者不断的从队列中获取消息，
因为消费者和生产者都是异步的，只关心消息的发送和接受，没有业务的侵入，别名消息中间件
```

```
MQ :ActiveMQ、Kafka、RabbitMQ、RockMQ
```

>RabbitMQ比Kafka可靠，Kafka更适合IO高吞吐的处理，一般应用于大数据日志处理或者对实时性（少量延迟）可靠性要求低的场景
>
>

```
生产者是发送消息的用户应用程序。

队列是存储消息的缓冲区。

使用者是接收消息的用户应用程序。

RabbitMQ消息传递模型的核心思想是生产者从不将任何消息直接发送到队列。生产者只能将消息发送到交换机。
```



# 一、hello-word

![模型展示](/Users/mac/Pictures/截图/屏幕快照 2021-04-19 15.13.15.png)

```
提供者向队列发送消息，
接受者从队列读取消息
消费者与提供者通过队列直接关联

发送完消息还是在队列中，直到有消费者消费到这条消息
消费者连接到队列的时候；立刻消费掉消息
```

##  1、配置rabbitMQ

```properties
# rabbitMQ配置
spring.rabbitmq.addresses=47.96.235.128:5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest

# 激活模型
spring.profiles.active=hello
```

## 2、配置

```java
@Configuration
@EnableConfigurationProperties(RabbitMQProperties.class)
public class RabbitMQConfig {

  	 @Autowired
    private RabbitMQProperties rabbitMQProperties;
  
  

    @Bean
    @Profile("hello")
    public Queue queue() {
        return new Queue("hello");
    }


    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        RabbitTemplate template = new RabbitTemplate(connectionFactory);
        template.setMessageConverter(new Jackson2JsonMessageConverter());
        return template;
    }


    @Bean
    public ConnectionFactory connectionFactory() {
        CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
        connectionFactory.setAddresses(rabbitMQProperties.getAddresses());
        connectionFactory.setUsername(rabbitMQProperties.getUsername());
        connectionFactory.setPassword(rabbitMQProperties.getPassword());
        return connectionFactory;
    }


    @Bean(value = "rabbitListenerContainerFactory")
    public RabbitListenerContainerFactory<?> rabbitListenerContainerFactory() {
        SimpleRabbitListenerContainerFactory simpleRabbitListenerContainerFactory = new SimpleRabbitListenerContainerFactory();
        // 消息转换
        simpleRabbitListenerContainerFactory.setMessageConverter(jackson2JsonMessageConverter());
        simpleRabbitListenerContainerFactory.setConnectionFactory(connectionFactory());
        return simpleRabbitListenerContainerFactory;
    }

    /**
     * 消息转换器
     * 将二进制消息转换成json消息
     * @return
     */
    @Bean
    public Jackson2JsonMessageConverter jackson2JsonMessageConverter() {
        return new Jackson2JsonMessageConverter();
    }

}
```

## 3、生产者

```java
@RestController
@RequestMapping("/amqp")
public class RabbitMQController {

  	# 在配置文件中指定的标准格式，就会自动组装成AmqpTemplate 对象
    # 如果想要自己书写AmqpTemplate 就需要在@Bean @Primy表明优先加载
    # 需要配置消息解析器
    @Autowired
    private AmqpTemplate amqpTemplate;

  # 队列
    @Autowired
    private Queue queue;


  
    /**
     * 使用hello模型发送消息
     */
    @GetMapping("/hell-world")
    public String helloWorld() {
      	#未填充的属性使用默认值
        amqpTemplate.convertAndSend(queue.getName(), "这是hello-word模型下的一条消息");
        return "success";
    }
}
```

## 4、消费者

```java
/**
 * @author by peilizhi
 * @date 2021/4/19 16:03
 * @RabbitListener 注解需要写在函数上，不要再写到类上
 */
@Component
public class HelloConsumer {
    /**
     * 处理消息
     * @param message
     */
    @RabbitListener(queues = "hello", containerFactory = "rabbitListenerContainerFactory")
    public void handle(Message message) {
        String body = new String(message.getBody());
        System.out.println("body = " + body);
        System.out.println("true = " + true);
    }
```



# 二、work模型

 <img src="/Users/mac/Pictures/截图/屏幕快照 2021-04-19 22.33.27.png" style="zoom:50%;" />

```
生产者直接将消息发送到队列（不设置交换机）
默认：队列将消息平均分配给消费者，采用自动确认机制，迅速返回结果，后续执行
	可以设置一次性分发给消费者的消息数量，PrefetchCount
可以设置能者多劳模型：手动确认消息（只有完成了才确认），此时必须需要设置每次分发给消费者的数量。
但如果设置分发的消息数大于队列中的总消息数，就采用平均分配的方式

一条消息仅能被一个消费者消费
```

```java
		@Bean(value = "work")
    public Queue workQueue(){
      # name：队列名称
      # durable：默认是true，持久化队列
      # exclusive：默认是false,独占队列
      # autoDelete默认是false ,消费完消息是否自动删除  
        return new Queue("work");
    }


		@Bean(value = "workRabbitListenerContainerFactory")
    public RabbitListenerContainerFactory<?> workRabbitListenerContainerFactory() {
        SimpleRabbitListenerContainerFactory simpleRabbitListenerContainerFactory = new SimpleRabbitListenerContainerFactory();
        // 消息转换
        simpleRabbitListenerContainerFactory.setMessageConverter(jackson2JsonMessageConverter());
        // 手动确认机制：默认是自动确认
        simpleRabbitListenerContainerFactory.setAcknowledgeMode(AcknowledgeMode.MANUAL);
        // 一次性分发给消费者的数量
        simpleRabbitListenerContainerFactory.setPrefetchCount(2);

        simpleRabbitListenerContainerFactory.setConnectionFactory(connectionFactory());
        return simpleRabbitListenerContainerFactory;
    }

```

## 、生产者

```java
@GetMapping("/work")
    public String work() {
        for (int i = 0; i < 10; i++) {
            rabbitTemplate.convertAndSend(workQueue.getName(), "这是work模型下的一条消息,编号:" + i);
        }
        return "work-success";
    }
```

## 3、消费者

```java
@Component
public class WorkAConsumer {
    /**
     * 处理消息
     * @param message
     * @param channel 通道
     */
    @RabbitListener(queues = "work", containerFactory = "workRabbitListenerContainerFactory")
    public void handle(Message message,Channel channel ) throws IOException {
        final long deliveryTag = message.getMessageProperties().getDeliveryTag();
        try {
            String body = new String(message.getBody());
            System.out.println("WorkAConsumer 接受到消息" + body);
            System.out.println("true = " + true);
            // deliveryTag消息编号
            // multiple:true 确定本消息编号之前的所有编号
            // false:仅确定本条消息
            channel.basicAck(deliveryTag,false);
        } catch (Exception e) {
            // 失败后重回队列
            channel.basicReject(deliveryTag,true);
        }
    }
}
```

>消费者可以在方法中加入与消费相关的参数，如果Message、Channel
>
>Channel 用于消息的确定机制。
>
>basicAck：用于肯定的回答，
>
>​	deliveryTag消息编号，
>
>​	multiple:   true 确定本消息编号之前的所有编号，false:仅确定本条消息
>
>basicReject:否定回答
>
>​	deliveryTag消息编号，
>
>​	requeue：失败后是否重回队列，false：不会再消费这条消息
>
>basicNack:否定回答
>
>​	deliveryTag：消息编号
>
>​	multiple： true 确定本消息编号之前的所有编号，false:仅确定本条消息
>
>​	requeue：失败后是否重回队列
>
>这三个方法都表示消息已被正确投递

# 三、publish/subscribe

![](/Users/mac/Pictures/截图/屏幕快照 2021-04-20 11.31.28.png)



```
生产者将消息发送非队列，队列再将消息发送给绑定到所有队列，及每个队列都接受到相同的消息。


交换机：它接收来自生产者的消息，另一方面，将它们推入队列。 交换机必须确切知道如何处理收到的消息。 是否应将其附加到特定队列？ 是否应该将其附加到许多队列中？ 还是应该将其丢弃。 规则由交换类型定义。


channel.basicPublish（“”，“ hello”，null，message.getBytes（））;

第一个参数是交换的名称。 空字符串表示默认或无名称交换：消息将以routingKey指定的名称路由到队列（如果存在）。
一个绑定对应一个队列，此时无需设置发送的对列，只需确定交换机的类型即可（交换机要向所有绑定的队列发送消息）
```

## 1、配置

```java
 /**
     * 广播类型交换机
     */
    @Bean
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("exchange.fanout");
    }

    @Bean
    Queue fanoutQueue1(){

        return QueueBuilder
                // 创建持久化队列
                .durable("fanout.queue1")
                .build();
    }

    @Bean
    Queue fanoutQueue2(){
        return QueueBuilder
                // 创建持久化队列
                .durable("fanout.queue2")
                .build();
    }

    @Bean
    Binding binding1(){
        // 绑定队列到交换机上
        return BindingBuilder
                .bind(fanoutQueue1())
                .to(fanoutExchange());
    }

    @Bean
    Binding binding2(){
        // 绑定队列到交换机上
        return BindingBuilder
                .bind(fanoutQueue2())
                .to(fanoutExchange());
    }

```

## 2、生产者

```java
 @GetMapping("/fanout")
    public String fanout() {
        for (int i = 0; i < 10; i++) {
            // 生产者发送消息的时候不需要关系发送到哪个队列中,无需指定队列
            rabbitTemplate.convertAndSend(fanoutExchange.getName(), null, "这是fanout模型下的一条消息,编号:" + i);
        }
        return "fanout-success";
    }
```

## 3、消费者

```java
@Component
public class FanoutAConsumer {
    @RabbitListener(queues = "fanout.queue1", containerFactory = "fanoutRabbitListenerContainerFactory")
    public void handle(Message message, Channel channel) throws IOException {
        final long deliveryTag = message.getMessageProperties().getDeliveryTag();
        try {
            String body = new String(message.getBody());
            System.out.println("FanoutAConsumer 接受到消息" + body);
            // deliveryTag消息编号
            // multiple:true 确定本消息编号之前的所有编号
            // false:仅确定本条消息
            channel.basicAck(deliveryTag, false);
        } catch (Exception e) {
            // 失败后重回队列
            channel.basicReject(deliveryTag, true);
        }
    }
}

```

# 四、topic







































二、注解实现zipkin链路监控（五一之后的第一周）

## 链路监控

