扫描流程 ->    Beandefinition处理   -> 创建bean（实例化） -> 属性赋值   -> 初始化   -> 销毁

# 扫描

<img src="https://cdn.jsdelivr.net/gh/Footman56/imageBed2/202411171230973.png" style="zoom:50%;" />

在判断includeFilters 时默认有一个配置就是配置类所在的包

# Beandefinition处理

<img src="https://cdn.jsdelivr.net/gh/Footman56/imageBed2/202411171231389.png" alt="image-20241117123119341" style="zoom:50%;" />



# 实例化

<img src="https://cdn.jsdelivr.net/gh/Footman56/imageBed2/202411280033580.png" alt="image-20241128003359515" style="zoom:50%;" />

@Annotation  数量问题

+ 一个required =true  ，并且其他required =false： 抛异常
+ 一个required =true  ，并且其他required =true :  抛异常
+ 允许多个required =false 的
+ 只有一个required =true

<img src="https://cdn.jsdelivr.net/gh/Footman56/imageBed2/202411280033392.png" alt="image-20241128003322221" style="zoom:50%;" />

# 属性赋值

<img src="https://cdn.jsdelivr.net/gh/Footman56/imageBed2/202411172259885.png" alt="image-20241117225950815" style="zoom:50%;" />

## Autowired

@Autowired 可以标注在构造方法、方法上、属性上、方法参数中、注解中。

本质都是从容器中找寻符合条件的bean，通过反射给属性设值

**由AutowiredAnnotationBeanPostProcessor 实现**



依赖注入原则-多值

1. @Value 的值，有的话就不会考虑下面

2. 先根据类型去找 ，筛选条件如下 【与当前bean 不同 beanName的】

   1. isAutowireCandidate=true 
   2. 如果是范型的话，需要符合bean 设置的范型
   3. 有@Qualifier 的话就需要名称保持一致，没有的话算符合条件

3. 如果单例池中有实例对象的话会采用。

4. 根据类型去找时没有符合条件的话，就将 当前bean 相同 beanName的纳入考虑条件再去找

   

依赖注入原则-单值

1. @Value 的值，有的话就不会考虑下面
2. 先根据类型去找   ，筛选条件如下【与当前bean 不同 beanName的】
   1. isAutowireCandidate=true 
   2. 如果是范型的话，需要符合bean 设置的范型
   3. 有@Qualifier 的话就需要名称保持一致，没有的话算符合条件
3. 如果单例池中有实例对象的话会采用。
4. 根据类型去找时没有符合条件的话，就将 当前bean 相同 beanName的纳入考虑条件再去找
5. 如果找出多个值的话
   1. 获取@Primary注解标注的【有多个报错】
   2. 获取@Priority 最小优先级高
   3. 单例池中存在的 或者 与参数名称相同的  获取到一个就返回【与BeanDefinitionNames顺序一致】

<img src="https://cdn.jsdelivr.net/gh/Footman56/imageBed2/202411180032555.png" alt="image-20241118003203495" style="zoom:50%;" />



@Resource 先根据name 去找，之后再根据类型去找

# 创建bean 、初始化

<img src="https://cdn.jsdelivr.net/gh/Footman56/imageBed2/202411171236465.png" alt="image-20241117123608424" style="zoom:50%;" />



dependentBeanMap:  类被哪些依赖 。 key：beanName  value:  所依赖beanName的所有类（哪些类中有beanName属性）

dependenciesForBeanMap： 类有哪些依赖。 key： beanName ， value: beanName的属性



实例化、初始化顺序

1. InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation

2. BeanPostProcessor#postProcessAfterInitialization

3. 实例化bean

4. MergedBeanDefinitionPostProcessor#postProcessMergedBeanDefinition

5. InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation 【实例化后】

6. InstantiationAwareBeanPostProcessor#postProcessProperties  

   1. AutowiredAnnotationBeanPostProcessor 实现 @Autowired、@Value、@Inject
   2. CommonAnnotationBeanPostProcessor 实现 @Resource

7. applyPropertyValues  【beanDefinition中属性赋值】

8. InstantiationAwareBeanPostProcessor#postProcessPropertyValues

9. 执行BeanNameAware、BeanClassLoaderAware、BeanFactoryAware对应方法

10. 执行BeanPostProcessor#postProcessBeforeInitialization 【初始化前】

    1. InitDestroyAnnotationBeanPostProcessor  实现 @PostConstruct、@PreDestroy
    2. ApplicationContextAwareProcessor 实现 Aware接口

11. InitializingBean#afterPropertiesSet

12. RootBeanDefinition中配置的InitMethod

13. BeanPostProcessor#postProcessAfterInitialization  【初始化后】

    

# 循环依赖

理论推导：

1. 实例化A ->  需要AOP 就创建代理对象  -> 存入早期对象池
2. 属性注入B -> 从单例池中获取-> 没有获取从早期对象池中获取 -> 创建B
3. 初始化前
4. 初始化
5. 初始化后 -> AOP



1. 实例化B
2. 属性注入A -> 从单例池中获取-> 没有获取从早期对象池中获取
3. 初始化前
4. 初始化
5. 初始化后

此时发生循环依赖



早期对象池的中对象与单例池中的对象是一个，保持相同的引用

问题：如果对象A需要进行AOP的话，单例池中的对象是代理对象。而B属性注入的是获取的A是原始半成品对象

答：解决方式就是早期对象池中的对象要么是代理对象，要么是原始对象。虽然可以通过在实例化之后进行AOP创建代理对象，但是会污染Bean的生命周期，并且在初始化还需要进行AOP处理。

问题：什么时候才是判断是否需要AOP的最佳时机？

答：在实例化B 进行属性注入A 的时候才知道有循环依赖。（A正在创建，B属性注入的时候需要A   【A->C->B->A】），所以创建代理A的时机也在B 属性注入的时候。在实例化的时候存入` 创建原始对象或者代理对象的动作` 也就是`三级缓存`

问题：如果多个对象依赖A的话，每次遇到循环依赖的时候都进行AOP的话，每次的代理对象不一致，无法保证单例

答：缓存创建好的原始对象或者代理对象 也就是`二级缓存`



1. 实例化A ->  标记A 正在创建 ->  存入 < AName, lamda(返回原始对象或者代理对象) >singletonObjects
2. 属性注入B  -> 从单例池中获取->  没有的话从二级缓存中获取 -> 没有的话调用singletonObjects 的lamda创建(标记执行过AOP)  -> 存入二级缓存
3. 初始化前
4. 初始化
5. 初始化后 -> AOP -> 是否执行过AOP -> 没有执行过就执行AOP



1. 实例化B
2. 属性注入A -> 从单例池中获取->  没有的话从二级缓存中获取 -> 没有的话调用singletonObjects 的lamda创建 -> 存入二级缓存
3. 属性注入C  -> 从单例池中获取->  没有的话从二级缓存中获取 -> 没有的话调用singletonObjects 的lamda创建 -> 存入二级缓存
4. 初始化前
5. 初始化
6. 初始化后



1. 实例化C
2. 属性注入A -> 从单例池中获取->  没有的话从二级缓存中获取【二级缓存就可以获取之前创建好,保证单例】-> 没有的话调用singletonObjects 的lamda创建 -> 存入二级缓存
3. 初始化前
4. 初始化
5. 初始化后

<img src="https://cdn.jsdelivr.net/gh/Footman56/imageBed2/202411290036178.png" alt="image-20241129003613133" style="zoom:50%;" />

循环依赖破局的关键在于从单例池中获取不到的时候 获取半成品的对象，避免一致等待创建



正常情况下spring能处理循环依赖，除非

+ 关闭允许循环依赖按钮
+ 循环依赖发生在构造器中【因为需要先获取实例】

**两个原型bean之间的循环依赖解决不了**，因为原型是每次创建的时候都需要创建新的Bean,不能使用之前创建好的bean



类A有@Async标志的方法会创建代理对象，注意在循环依赖中如果先创建的类A,会报错，因为@Async是在AOP之后创建的代理，所以容器中的因@Async创建的代理，但是别的类依赖的属性是AOP代理，所以有问题

@Transate 实现不是创建代理类，所以不影响



解决代理类最好的方式就是加 @Lazy 注解。



# 销毁



<img src="https://cdn.jsdelivr.net/gh/Footman56/imageBed2/202411171716430.png" style="zoom: 33%;" />

bean 注销的时机为 容器关闭时】并且只需要销毁单例bean

符合bean销毁的情景

+ 不是多例 bean

+  实现 DisposableBean    

+ Beandefinition中定义了destroyMethodName 

+ Beandefinition中destroyMethodName = ‘“(inferred)”  并且有 close()  或者有shutdown()

+ 实 现AutoCloseable 中close 方法

+ DestructionAwareBeanPostProcessor#requiresDestruction 返回true

  + 这就是@PreDestroy功能的实现

    

**disposableBeans 采用适配器模式，将上面符合条件的多种情况都转换成DisposableBeanAdapter ，将多种情况的销毁方法，都统一成一个。**



经验：

+ 在bean的生命周期中都提供了扩展的插件  思路为 BeanPostProcessor 中的各个接口
+ 收集了每个一环节的结果，并广泛使用缓存，类似：singletonObjects
+ 在类中可以注入多个工具类，用于处理对应的功能。将任务分发出去