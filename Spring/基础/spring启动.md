一、 根据配置类创建成AnnotatedGenericBeanDefinition,并且缓存到beanDefinitionMap

二、 扫描 -> 生成beanDefinition【是在ConfigurationClassPostProcessor 后置处理器中处理的】

1. 是否有spring索引，如果有的话，只解析索引里面的bean，创建Set<BeanDefinition>
2. 获取包下所有资源
3. 解析资源成元信息（类上是否有注解 ，有哪些注解）
4. 结合信息中inclueFilters （默认包含@compoent 注解）,exclueFilters， 包含并且是否符合条件@Conditional
5. 符合条件的创建成ScannedGenericBeanDefinition，beanClass = beanClassName，后面会设置成class
6. 非抽象类、非接口等加入到Set<BeanDefinition>
7. 解析@Lazy、@Primary、@DependsOn、@Role、@Description
8. 检查容器中是否有相同名称的beanNames ，有重复但是不是命名空间的的，抛异常
9. 注册到beanDefinitionMap，aliasMap

三、根据beanDefinition 创建非懒加载单例bean

1. 遍历已经收集好的beanDefinition

2. 创建RootBeanDefinition （合并后的bean）一般注解生成的就没有父bean， xml文件配置的会有，要合并成一个RootBeanDefinition,因为子类会用父类的属性

3. 判断是否要创建单例（非接口、非抽象类等）

4. 如果是的BeanFactory，不光要创建BeanFactory实例，也要创建BeanFactory里面定义的实例（仅限于SmartFactoryBean） ,普通的BeanFactory 里面定义的实例是在获取的时候创建，不是在容器启动的时候创建

5. 如果不是BeanFactory 就要创建，创建后放入单例池中

   1. 获取合并后的RootBeanDefinition

   2. 是否有@dependsOn，有的话先创建依赖bean

   3. 是否是单例，原型、request、session 类型的

   4. 获取类加载器，由类加载获取class

   5. 先执行父类方法

   6. **InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation**，用于自定义根据class 创建实例，循环执行，直到终止循环或者有实例被创建

   7. 如果经**InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation** 创建好实例之后再执行**BeanPostProcessor#postProcessAfterInitialization**，退出

   8. 如果RootBeanDefinition 中定义了Supplier ，用Supplier 创建实例，退出

   9. 如果有工厂方法，直接执行工厂方法，退出

   10. 找寻合适的构造方法，创建实例

   11. **MergedBeanDefinitionPostProcessor#postProcessMergedBeanDefinition**，用于根据RootBeanDefinition对实例进行处理，可以设置初始化方法

   12. **InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation**，对实例进行自定义设置值，返回false 退出

   13. 处理@Bean(autowire = Autowire.BY_NAME) 这种形式的，指定怎么获取属性【autowire已废弃，xml 扔可用】

   14. **InstantiationAwareBeanPostProcessor#postProcessProperties**，@Autowired 注解实现

   15. 如果当前Bean中的BeanDefinition中设置了PropertyValues，那么最终将是PropertyValues中的值，覆盖@Autowired

   16. Aware 接口实现处理

   17. **BeanPostProcessor#postProcessBeforeInitialization**，其中InitDestroyAnnotationBeanPostProcessor 实现了@PostConstruct、@PreDestroy

   18. **InitializingBean#afterPropertiesSet**

   19. 执行RootBeanDefinition中配置的initMethodName

   20. **BeanPostProcessor#postProcessAfterInitialization**

       

