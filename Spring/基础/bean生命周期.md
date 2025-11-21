Class ---->  构造方法  ---->  普通对象  ---->  依赖注入  ----> 初始化前（@PostConstruct） ---->  初始化（InitializingBean）  ---->  初始化后（AOP）  ---->  **代理对象**  ----> bean 

1. 首先是执行构造方法

   会使用默认的构造方法，如果没有的话，就使用唯一的方法，如果有多个的话，Spring 就不知道使用哪个对象

   可以使用@Autowired 来执行具体的构造方法

2. 如果构造方法中有参数，执行时这个参数是有值



怎么给构造方法的参数赋值呢？

```
通过容器中对象给参数赋值
```

怎么从容器中拿取对象呢？

```
a. 首先获取容器中相同类的对象
b. 如果仅有一个就使用这个
c. 如果有多个的话，就根据名称获取
b. 如果都没有获取的话，就报错
```

容器中的对象，可能是实例对象，也有可能是代理对象，取决于是否执行过AOP

```java
@Component
public class UserService {
    @Autowired
    private OrderService orderService;
    public void save() {
        System.out.println("orderService = " + orderService);
    }
}
// 这种方法创建处理的UserService 是实例，没有经过代理
```

![image-20241107230412562](https://cdn.jsdelivr.net/gh/Footman56/imageBed2/202411072304933.png)



```java
@Component
@Aspect
public class BeanBefore {
    @Before( "execution(public void com.huochai.demos.service.UserService.save())")
    public void setup() {
        System.out.println("before");
    }
}

```

```java
 // 需要开启EnableAspectJAutoProxy
@EnableAspectJAutoProxy
@Configuration
@ComponentScan("com.huochai.demos")
public class BeanConfig {
}

```

![image-20241107232748692](https://cdn.jsdelivr.net/gh/Footman56/imageBed2/202411072327757.png)

1. 在配置文件将所有切面加入到容器中
2. 获取所有切面
3. 遍历切面里面的所有切入点
4. 将切入点与通知缓存起来
5. 在创建bean 时候检查是否在 切入点中，如果在的话就创建代理对象，否则就是普通对象



因为容器中存的是代理对象，使用cglib 代理时会 代理对象为目标对象的子类。所以代理对象就可以执行对应的方法，执行方式的时候 先执行通知，之后调用target来执行目标方法【target是原始对象】



事务使用的datascoure 必须要与实际操作数据的 datascoure 是同一个，否则就无法保证事务

事务失效的问题？调用本类里面的方法（方法上通过@Transtional）









BeanFactory  与 FactoryBean 的区别？

BeanFactory 负责bean的创建，简单工厂模式

FactoryBean 可以修改创建bean的逻辑，通过重载getObject()、 getObjectType()  可以修改容器中创建bean的逻辑，容器中通过beanName获取到的是getObject() 方法返回，想要获取原始对象， 可以 &beanName 方式。





AnnotationConfigApplicationContext 与BeanFactory 的区别？

BeanFactory 是工厂，只负责创建bean

AnnotationConfigApplicationContext 是BeanFactory 的接口实现，是BeanFactory 获取bean的自动化封装，并且提供了额外的功能，

两者都可以作为容器，能获取bean。
