BeanDefinition

用于描述 bean 的类名、是否单例、初始化、销毁方法。统一管理容器内的bean

# 功能

**定义 Bean 的类** 用于指定要实例化的 Bean 的类名。它告诉 Spring 容器要创建哪个 Java 类的对象。

**定义 Bean 的作用域** 允许我们指定 Bean 的作用域，例如 singleton（单例）或 prototype（多例）。这影响了 Bean 在容器中的生命周期。

**构造函数参数和属性值**  允许我们指定 Bean 的构造函数参数和属性值，以便在实例化 Bean 时传递参数或设置属性。

**定义初始化和销毁方法**  定义 Bean 的初始化方法和销毁方法，以确保在 Bean 创建和销毁时执行特定的逻辑。

**Bean 的延迟初始化**  允许我们设置 Bean 是否延迟初始化，即在第一次请求时创建 Bean 实例。

**依赖关系**  允许我们指定 Bean 之间的依赖关系，以确保在创建 Bean 时正确注入依赖的其他 Bean。

**描述 Bean 的角色**  允许我们为 Bean 指定一个角色（role），通常包括应用程序 Bean、基础设施 Bean、测试 Bean 等。

**Bean 的属性覆盖**  允许我们使用属性覆盖机制，通过不同的配置源（如属性文件或环境变量）覆盖已定义的属性值。

**Bean 的注解和元数据**  可以包含关于 Bean 的注解信息和元数据，这对于处理注解驱动的开发非常有用。

**动态创建和注册 Bean**  允许我们在运行时动态创建和注册 Bean，而不仅仅是静态配置。

# 实现

```java
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {
  
  void setParentName(@Nullable String parentName);
  
  void setBeanClassName(@Nullable String beanClassName);
}
```

# 子类

+ GenericBeanDefinition : 比较通用的实现，具有灵活的属性配置，可以设置类名、作用域、初始化和销毁方法、构造函数参数、属性值等。**通常用于手动配置 Bean 或需要自定义 Bean 定义的情况。**
+ RootBeanDefinition： 用于表示独立的根级 Bean 定义，通常直接定义 Bean。通常用于定义应用程序中的独立 Bean。
+ AnnotatedGenericBeanDefinition：用于基于注解的 Bean 定义，通常用于扫描组件和配置类。 可以表示使用注解定义的 Bean，支持类级别的注解配置。**通常用于自动扫描和注册组件**。 
+ ConfigurationClassBeanDefinition：用于表示配置类（`@Configuration` 注解）的 Bean 定义，通常用于 Spring 配置。表示配置类作为 Bean 定义，支持包含其他 Bean 定义的配置类。**通常由 Spring 容器自动创建**，以支持 `@Configuration` 注解的配置。
+ ScannedGenericBeanDefinition： 用于表示扫描到的 Bean 的 Bean 定义，通常用于自动扫描组件。通常在组件扫描过程中创建，表示被发现的组件类。**通常用于自动注册组件**，如 `@Component` 注解的类。

# 使用

```java
public static void main(String[] args) throws Exception {
  // 用于注册bean
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        beanFactory.registerBeanDefinition("myBean", createBeanDefinition());

        // 获取MyBean
        MyBean myChildBean = beanFactory.getBean("myBean", MyBean.class);
        // 打印Bean对象
        System.out.println("MyBean = " + myChildBean);
        // 销毁myBean
        beanFactory.destroySingleton("myBean");
    }

    private static BeanDefinition createBeanDefinition() throws IOException {
        SimpleMetadataReaderFactory metadataReaderFactory = new SimpleMetadataReaderFactory();
      // 获取数据元信息
        MetadataReader metadataReader = metadataReaderFactory.getMetadataReader(MyBean.class.getName());

        ScannedGenericBeanDefinition beanDefinition = new ScannedGenericBeanDefinition(metadataReader);
        beanDefinition.setScope("singleton");
        beanDefinition.setLazyInit(true);
        beanDefinition.setPrimary(true);
        beanDefinition.setAbstract(false);
        beanDefinition.setInitMethodName("init");
        beanDefinition.setDestroyMethodName("destroy");
        beanDefinition.setAutowireCandidate(true);
        beanDefinition.setRole(BeanDefinition.ROLE_APPLICATION);
        beanDefinition.setDescription("This is a custom bean definition");
        beanDefinition.setResourceDescription("com.xcs.spring.BeanDefinitionDemo");
        beanDefinition.getPropertyValues().add("name", "lex");
        beanDefinition.getPropertyValues().add("age", "18");
        return beanDefinition;
    }
```



# 注意

**`DefaultListableBeanFactory `** 负责管理Bean的创建、初始化和销毁，而`BeanDefinition`提供了描述Bean的元信息的方式，`DefaultListableBeanFactory`使用`BeanDefinition`来创建和管理Bean实例。



`GenericApplicationContext` 是`DefaultListableBeanFactory`的更高级别扩展，它不仅实现了 `DefaultListableBeanFactory` 的功能，还提供了应用程序级别的服务，如国际化、事件发布、资源加载等。 `DefaultListableBeanFactory` 通常使用 `BeanDefinitionRegistry` 来注册和管理Bean定义，以便在应用程序上下文中配置和管理Bean。



**`BeanPostProcessor`** : 拦截Bean初始化过程的接口，它可以在Bean创建后、初始化前后对Bean进行处理。`BeanDefinition`的信息可以在`BeanPostProcessor`中使用，例如在初始化前修改Bean的属性值。

**`BeanDefinitionRegistry`**  :注册和管理`BeanDefinition`的接口，定义了`BeanDefinition`的注册和访问方法。`BeanFactory`和`ApplicationContext`实现了`BeanDefinitionRegistry`接口，通过它们可以注册和获取`BeanDefinition`。

**`BeanDefinitionReader`**  从外部配置文件（如`XML、YAML、Properties`文件）中读取`BeanDefinition`的工具。它将外部配置信息解析成`BeanDefinition`并注册到`BeanFactory`中。





# 常见问题

如何使用`BeanDefinition`来实现依赖注入？

```
通过设置`BeanDefinition`中的属性，如构造函数参数、属性值、引用其他Bean的方式，可以实现依赖注入。
```

**`BeanDefinition`的注册和加载有什么区别？**

```
注册是将已创建的`BeanDefinition`添加到容器中，而加载是从外部配置文件中读取Bean的元信息并注册到容器中。
```
