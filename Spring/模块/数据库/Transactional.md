使用@Transactional方法来声明事物。

特点是简单方便，不足是无法控制细粒度。只能对方法进行注解。



如果方法比较复杂，实现的功能比较耗时，就可能导致`CannotGetJdbcConnectionException`**，数据库连接池连接占满。**

长事务引发的常见危害有：

1. 数据库连接池被占满，应用无法获取连接资源；

2. 容易引发数据库死锁；

3. 数据库回滚时间长；

4. 在主从架构中会导致主从延时变大。

   

原因是：使用@Transactional 包裹的整个方法都是使用同一个connection连接，长期占用链接的话，如果有大量的s q l请求的话，可能就无法为新的请求分配connection

解决方法：控制@Transactional 包裹方法尽可能的保证小粒度，执行不耗时

或者

在复杂的方法内部使用编程式事物，手动声明、开启、关闭。

```java
@Autowired 
private TransactionTemplate transactionTemplate; 
 
... 

public void save(RequestBill requestBill) { 
  // 声明事物
    transactionTemplate.execute(transactionStatus -> {
        requestBillDao.save(requestBill);
        //保存明细表
        requestDetailDao.save(requestBill.getDetail());
        return Boolean.TRUE; 
    });
} 
```

可以对业务进行拆分，将不需要事务管理的逻辑与事务操作分开：

但是 **注意要使@Transactional  生效**

`@Transactional`注解的声明式事务是通过spring aop起作用的，而spring aop需要生成代理对象，直接在同一个类中方法调用使用的还是原始对象，事务不生效



- @Transactional 应用在非 public 修饰的方法上
- @Transactional 注解属性 propagation 设置错误
- @Transactional 注解属性 rollbackFor 设置错误
- **同一个类中方法调用，导致@Transactional失效**
  - 只要是本类中调用本方法上有 异步，缓存，事务注解的，都使用自注入self来调也可以
- 异常被catch捕获导致@Transactional失效





想用本类中调用另一个被@Transactional 包裹方法可以

启动类添加`@EnableAspectJAutoProxy(exposeProxy = true)`，方法内使用`AopContext.currentProxy()`获得代理类

```java
SpringBootApplication.java

@EnableAspectJAutoProxy(exposeProxy = true)
@SpringBootApplication
public class SpringBootApplication {}




OrderService.java
  
public void createOrder(OrderCreateDTO createDTO){
    OrderService orderService = (OrderService)AopContext.currentProxy();
    orderService.saveData(createDTO);
}
```







长事物可以使用Sagas模式？