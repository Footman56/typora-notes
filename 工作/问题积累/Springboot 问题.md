1. com.github.pagehelper.autoconfigure.PageHelperAutoConfiguration 循环依赖

   之前在项目中引入的

   ```
   <dependency>
               <groupId>com.github.pagehelper</groupId>
               <artifactId>pagehelper-spring-boot-starter</artifactId>
               <version>1.2.7</version>
           </dependency>
   ```

   启动的时候

   ![image-20230317105909732](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202303171059200.png)

解决方式就是升级版本 1.4.1 之后再启动就成功啦



如果编译的时候提示某个类找不到或者对应的参数类型不正确的话。可以先删除本地client jar 之后再重新depolyment





2. 如果工程里面没有spring-boot-starter-web 依赖的话，启动之后会立刻销毁



3. @Transactional事务中发送MQ消息，事务还未完成但是消息已经发送

   事务未提交的时候成功发功消息，在消息处理的时候，可能与数据库中的数据不一致（事务未提交）。

   解决方式：在事务提交之后再消费消息

   ```java
   @Component
   public class AmqpTemplateHelper {
    
       @Autowired
       private AmqpTemplate amqpTemplate;
    
       public <T> void send(String queue, T message) {
           // 是否开启事务判断
           if (TransactionSynchronizationManager.isSynchronizationActive()) {
               TransactionSynchronizationManager.registerSynchronization(new TransactionSynchronizationAdapter() {
                   @Override
                   public void afterCommit() {
                       amqpTemplate.convertAndSend(queue, message);
                   }
               });
           } else {
               amqpTemplate.convertAndSend(queue, message);
           }
       }
   }
   ```

4.  在事务中需要发布一个事件，如何保证事件能够获取到事务修改的数据

   + 发布同步事件，并且保证同步事件与当前事务处于同一个事务中 

     ```java
     @Service
     public class MyService {
     
         @Autowired
         private ApplicationEventPublisher eventPublisher;
     
         @Transactional
         public void updateDataAndPublishEvent() {
             // 修改数据
             updateData();
     
             // 发布同步事件
             eventPublisher.publishEvent(new MyCustomEvent(this));
     
             // 此时事务还未提交
         }
     }
     
     @Component
     public class MyEventListener {
     
         @EventListener
         @Transactional(propagation = Propagation.REQUIRED)
         public void handleMyCustomEvent(MyCustomEvent event) {
             // 能够访问未提交的事务修改的数据
             accessData();
         }
     }
     ```

     
