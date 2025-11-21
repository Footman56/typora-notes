# 一、发送消息确认

<img src="/Users/mac/Pictures/截图/屏幕快照 2021-05-09 10.39.35.png" alt="消息流程" style="zoom:50%;" />

```
消息确认分为两部分：
	发送方的消息确认
	接受方的消息确认
```

<img src="/Users/mac/Pictures/截图/屏幕快照 2021-05-09 10.48.17.png" style="zoom:50%;" />



## 0、 执行流程

#### a、引入配置文件

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>

```

####  b 、配置

```properties
# 连接配置
spring.rabbitmq.host=127.0.0.1
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest

# 发送者开启 confirm 确认机制
spring.rabbitmq.publisher-confirms=true
# 发送者开启 return 确认机制
spring.rabbitmq.publisher-returns=true
####################################################
# 设置消费端手动 ack
spring.rabbitmq.listener.simple.acknowledge-mode=manual
# 是否支持重试
spring.rabbitmq.listener.simple.retry.enabled=true

```

#### c、Config

```java
/***
* 绑定关系
**/
@Configuration
public class BingConfig {

    @Bean(name = "confirmTestQueue")
    public Queue confirmTestQueue() {
        return new Queue("confirm_test_queue", true, false, false);
    }

    @Bean(name = "confirmTestExchange")
    public FanoutExchange confirmTestExchange() {
        return new FanoutExchange("confirmTestExchange");
    }

    @Bean
    public Binding confirmTestFanoutExchangeAndQueue(
            @Qualifier("confirmTestExchange") FanoutExchange confirmTestExchange,
            @Qualifier("confirmTestQueue") Queue confirmTestQueue) {
        return BindingBuilder.bind(confirmTestQueue).to(confirmTestExchange);
    }
}
```

**根据标准的配置项可以不用指定发送的工具，连接工厂等信息**

#### d、 ConfirmCallback确认模式

```
消息只要被 rabbitmq broker 接收到就会触发 confirmCallback 回调 。
实现接口 ConfirmCallback ，重写其confirm()方法，方法内有三个参数correlationData、ack、cause。

correlationData：对象内部只有一个 id 属性，用来表示当前消息的唯一性。
ack：消息投递到broker 的状态，true表示成功。
cause：表示投递失败的原因。
```

```java
@Slf4j
@Component
public class ConfirmCallbackService implements RabbitTemplate.ConfirmCallback {
    
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {

        if (!ack) {
            log.error("消息发送异常!");
        } else {
            log.info("发送者爸爸已经收到确认，correlationData={} ,ack={}, cause={}", correlationData.getId(), ack, cause);
        }
    }
}
```

#### e、ReturnCallback 退回模式

```
如果消息未能投递到目标 queue 里将触发回调 returnCallback ，一旦向 queue 投递消息未成功，这里一般会记录下当前消息的详细投递数据，方便后续做重发或者补偿等操作

实现接口ReturnCallback，重写 returnedMessage() 方法，
方法有五个参数message（消息体）、replyCode（响应code）、replyText（响应内容）、exchange（交换机）、routingKey（队列）。

```

```java
@Slf4j
@Component
public class ReturnCallbackService implements RabbitTemplate.ReturnCallback {

    @Override
    public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
        log.info("returnedMessage ===> replyCode={} ,replyText={} ,exchange={} ,routingKey={}", replyCode, replyText, exchange, routingKey);
    }
}
```

#### f、 发送消息示例

```java
@Autowired
    private RabbitTemplate rabbitTemplate;

    @Autowired
    private ConfirmCallbackService confirmCallbackService;

    @Autowired
    private ReturnCallbackService returnCallbackService;

    public void sendMessage(String exchange, String routingKey, Object msg) {

        /**
         * 确保消息发送失败后可以重新返回到队列中
         * 注意：yml需要配置 publisher-returns: true
         */
        rabbitTemplate.setMandatory(true);

        /**
         * 消费者确认收到消息后，手动ack回执回调处理
         */
        rabbitTemplate.setConfirmCallback(confirmCallbackService);

        /**
         * 消息投递到队列失败回调处理
         */
        rabbitTemplate.setReturnCallback(returnCallbackService);

        /**
         * 发送消息
         */
        rabbitTemplate.convertAndSend(exchange, routingKey, msg,
                message -> {
                    message.getMessageProperties().setDeliveryMode(MessageDeliveryMode.PERSISTENT);
                    return message;
                },
                new CorrelationData(UUID.randomUUID().toString()));
    }

```

#### g 、接受消息确认

```java
@Slf4j
@Component
@RabbitListener(queues = "confirm_test_queue")
public class ReceiverMessage1 {
    
    @RabbitHandler
    public void processHandler(String msg, Channel channel, Message message) throws IOException {

        try {
            log.info("小富收到消息：{}", msg);

            //TODO 具体业务
            
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);

        }  catch (Exception e) {
            
            if (message.getMessageProperties().getRedelivered()) {
                
                log.error("消息已重复处理失败,拒绝再次接收...");
                
                channel.basicReject(message.getMessageProperties().getDeliveryTag(), false); // 拒绝消息
            } else {
                
                log.error("消息即将再次返回队列处理...");
                
                channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, true); 
            }
        }
    }
}

```

```
prefetch允许为每个consumer指定最大的unacked messages数目。简单来说就是用来指定一个consumer一次可以从Rabbit中获取多少条message并缓存在client中(RabbitMQ提供的各种语言的client library)。一旦缓冲区满了，Rabbit将会停止投递新的message到该consumer中直到它发出ack。
```

##  1.1 发送方的消息确认

### 1.1.1  背景

```shell
发送消息确认：用来确认生产者 producer 将消息发送到 broker ，broker 上的交换机 exchange 再投递给队列 queue的过程中，消息是否成功投递。
消息从 producer 到 rabbitmq broker有一个 confirmCallback 确认模式。
消息从 exchange 到 queue 投递失败有一个 returnCallback 退回模式。
```

### 1.1.2  策略

####  a、AMQP事务机制

```shell
RabbitMQ中与事务机制有关的方法有三个，分别是Channel里面的txSelect()，txCommit()以及txRollback().
txSelect用于将当前Channel设置成是transaction模式，txCommit用于提交事务，txRollback用于回滚事务，
在通过txSelect开启事务之后，我们便可以发布消息给broker代理服务器了，如果txCommit提交成功了，则消息一定是到达broker了，
如果在txCommit执行之前broker异常奔溃或者由于其他原因抛出异常，这个时候我们便可以捕获异常通过txRollback回滚事务了；
```

```
事务限制了AMQP执行的性能
```

#### b、confirm模式

```
生产者将信道设置成confirm模式，一旦信道进入confirm模式，所有在该信道上面发布的消息都将会被指派一个唯一的ID(从1开始)，
一旦消息被投递到所有匹配的队列之后，broker就会发送一个确认给生产者(包含消息的唯一ID)，这就使得生产者知道消息已经正确到达目的队列了，如果消息和队列是可持久化的，那么确认消息会在将消息写入磁盘之后发出，
broker回传给生产者的确认消息中delivery-tag域包含了确认消息的序列号，此外broker也可以设置basic.ack的multiple域，表示到这个序列号之前的所有消息都已经得到了处理；


confirm模式最大的好处在于他是异步的，一旦发布一条消息，生产者应用程序就可以在等信道返回确认的同时继续发送下一条消息，当消息最终得到确认之后，生产者应用便可以通过回调方法来处理该确认消息，
如果RabbitMQ因为自身内部错误导致消息丢失，就会发送一条nack消息，生产者应用程序同样可以在回调方法中处理该nack消息；
```

#### c、比较

```
事务模式与confirm模式不能同时存在

(1)：普通confirm模式，每发送一条消息，调用waitForConfirms()方法等待服务端confirm，这实际上是一种串行的confirm，每publish一条消息之后就等待服务端confirm，如果服务端返回false或者超时时间内未返回，客户端进行消息重传；
(2)：批量confirm模式，每发送一批消息之后，调用waitForConfirms()方法，等待服务端confirm，这种批量确认的模式极大的提高了confirm效率，但是如果一旦出现confirm返回false或者超时的情况，客户端需要将这一批次的消息全部重发，这会带来明显的重复消息，如果这种情况频繁发生的话，效率也会不升反降；
```

#### d、后续操作

```
生产者接受到broker发送的确认消息，可以通过在通道中绑定监听器的方法来对确认消息进行处理。
要拿到这个ack消息进行后期操作。
要想拿到ack消息的话，我们可以给当前Channel信道绑定监听器，
具体来说就是调用Channel信道的addConfirmListener方法进行设置，Channel信道在收到broker的ack消息之后会回调设置在该信道监听器上的handleAck方法，在收到nack消息之后会回调设置在该信道监听器上的handleNack方法。
```

## 1.2 接受方的消息确认

### 1.2.1 消息回执

#### a、basicAck

```
basicAck：表示成功确认，使用此回执方法后，消息会被rabbitmq broker 删除。
void basicAck(long deliveryTag, boolean multiple) ;
	deliveryTag：表示消息投递序号，每次消费消息或者消息重新投递后，deliveryTag都会增加。手动消息确认模式下，我们可以对指定deliveryTag的消息进行ack、nack、reject等操作。
	multiple：是否批量确认，值为 true 则会一次性 ack所有小于当前消息 deliveryTag 的消息。
```

#### b、baskNack

```
basicNack ：表示失败确认，一般在消费消息业务异常时用到此方法，可以将消息重新投递入队列。
void basicNack(long deliveryTag, boolean multiple, boolean requeue);
	deliveryTag：表示消息投递序号。
	multiple：是否批量确认。
	requeue：值为 true 消息将重新入队列。
```

#### c、basicReject

```	
basicReject：拒绝消息，与basicNack区别在于不能进行批量操作，其他用法很相似。
void basicReject(long deliveryTag, boolean requeue)
	deliveryTag：表示消息投递序号。
	requeue：值为 true 消息将重新入队列。
```

### 1.2.2 消息无限投递

```
在消费者处理完业务逻辑的时候发生异常，之后会将这条消息再次投递到消费队列，之后消费者还要再次消费，重复报错。
当消息重新投递到消息队列时，这条消息不会回到队列尾部，仍是在队列头部。

解决方案可以是：
	在消息执行异常时，将消息不在投递到消费队列，之后再次发起这条消息
	设置了消息重试次数，达到了重试上限以后，手动确认，队列删除此消息，并将消息持久化入MySQL并推送报警，进行人工处理和定时任务做补偿。
```







