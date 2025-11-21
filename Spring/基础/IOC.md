```
IoC 也称为依赖注入 (DI)。 这是一个过程，其中对象定义了它们的依赖关系，即它们使用的其他对象，仅通过 构造函数参数、工厂方法的参数或在对象实例被构造 或从工厂方法返回后在对象实例上设置的属性 . 然后容器在创建 bean 时注入这些依赖项。

BeanFactory 接口提供了一种能够管理任何类型对象的高级配置机制。 ApplicationContext 是 BeanFactory 的一个子接口。

Bean 以及它们之间的依赖关系反映在容器使用的配置元数据中。 这些元数据以xm和注解的形式、java代码存在。
```

![](/Users/peilizhi/screenshots/截屏2021-09-11 19.23.45.png)

# 初识容器

将bean和配置元信息加入到容器中，之后就可以获取到我们需要使用到bean

```
Spring 2.5 引入注解形式的元信息
Spring 3.0 支持java 代码形式。@Configuration, @Bean, @Import and @DependsOn annotations.
```

```
容器中的bean通常为服务层对象、数据访问对象 (DAO)、表示对象（例如 Struts Action 实例）、基础设施对象（例如 Hibernate SessionFactories、JMS 队列等）。不配置细粒度的对象属性。可通过AspectJ来定义对象属性
```

xml形式的bean定义

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">
  
  <!--使用一个或多个 <import/> 元素从另一个文件或多个文件加载 bean 定义  
所有位置路径都相对于执行导入的定义文件，因此 services.xml 必须与执行导入的文件位于同一目录或类路径位置  
前导斜杠被忽略--->
  <import resource="/resources/themeSource.xml"/>


  
<!--id 属性是一个字符串，用于标识单个 bean 定义。--->
  <bean id="..." class="...">
    <!-- collaborators and configuration for this bean go here -->
  </bean>

  <!--property name 元素是指 JavaBean 属性的名称，ref 元素是指另一个 bean 定义的名称。-->
  <bean id="..." class="...">
    <!-- collaborators and configuration for this bean go here -->
    <property name="accountDao" ref="accountDao"/>
    <property name="itemDao" ref="itemDao"/>
  </bean>
  
  
  <!-- more bean definitions go here -->
</beans>
```

```
实例化 Spring IoC 容器 提供给 ApplicationContext 构造函数的位置路径或路径实际上是资源字符串，允许容器从各种外部资源（例如本地文件系统、Java CLASSPATH 等）加载配置元数据。

ApplicationContext context =
    new ClassPathXmlApplicationContext(new String[] {"services.xml", "daos.xml"});
```

```
ApplicationContext 是高级工厂的接口，能够维护不同 bean 及其依赖项的注册表。 使用方法 
T getBean(String name, Class<T> requiredType) 可以检索 bean 的实例。

// create and configure beans
ApplicationContext context =
    new ClassPathXmlApplicationContext(new String[] {"services.xml", "daos.xml"});

// retrieve configured instance
PetStoreServiceImpl service = context.getBean("petStore", PetStoreServiceImpl.class);

应用程序最好不要使用这种形式获取bean，根本没有对Spring API的依赖性。直接通过注解依赖注入对象
```

```
在ioc 容器中，bean的 定义信息被当作 BeanDefinition 这个对象。
包含了： 
	包限定的类名：通常是定义的 bean 的实际实现类。
	Bean 行为配置元素，它说明 Bean 在容器中的行为方式（范围、生命周期回调等）。
	对 bean 执行其工作所需的其他 bean 的引用； 这些引用也称为协作者或依赖项。 依赖项
	要在新创建的对象中设置的其他配置设置
```

```
支持用户注册容器外的 已创建的对象。（可以向容器手动注册一些对象）

这是通过ApplicationContext.getBeanFactory()方法访问ApplicationContext的BeanFactory
返回的BeanFactory实现DefaultListableBeanFactory。DefaultListableBeanFactory通过registerSingleton(..)和registerBeanDefinition(..)方法支持此注册。
```

```
在容器中，每个bean的id 唯一，默认可以不指定。一个 bean 通常只有一个标识符，但如果它需要多个标识符，则可以将多余的标识符视为别名。
如果在别的bean中当前bean 作为属性被引用，就需要指定bean的名称
```

**如果在xml使用id 或者name来定义bean. 从容器中获取的时候也可以根据name来获取bean**

```
<alias name="subsystemA-dataSource" alias="myApp-dataSource" /> 定义别名。
别名的作用是同一个对象，在不同的作用域有着不同的名称，但本质还是同一个对象
name 属性指的是原来的名称， alias 指的是现在的名称
```

```
实例化bean 的本质是通过 定义好的bean配置信息来创建对象。在bean定义的时候需要指明类型。否则可以使用工厂方法来构造bean
```

实例化bean 的方法：

1、根据定义中bean的类型，通过反射创建对象。本质还是使用new方法来创建

2、指定能够调用静态工厂方法来创建对象的工厂类。使用工厂方法来创建对象

```
在使用xml形式配置bean的时候，如果类含有静态内部类的话，class 中需要使用$来分格内部类和外部类
```

```java
<bean id="outClass" class="com.huochai.bean.domain.OutClass">
        <property name="name" value="lizhi"/>
    </bean>

    <bean id="innerClass" class="com.huochai.bean.domain.OutClass$InnerClass">
        <property name="name" value="zhangya"/>
        <property name="old" value="21"/>
    </bean>
    
    public class OutClass {
    private String name;

    public void setName(String name) {
    }


    public static class InnerClass {
        private String name;
        private Integer old;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public Integer getOld() {
            return old;
        }

        public void setOld(Integer old) {
            this.old = old;
        }

        @Override
        public String toString() {
            return "InnerClass{" +
                    "name='" + name + '\'' +
                    ", old=" + old +
                    '}';
        }
    }
}

```

**通过xml指定bean的属性的时候，需要提供相应的get方法**



可以通过在be an定义中添加工厂类和工厂方法来创建实例

```xml
<!--    工厂类-->
    <bean id="suppliesFactory" class="com.huochai.bean.domain.SuppliesFactory"/>

    <!--此时不需要指定class属性-->
    <bean id="pen" factory-bean="suppliesFactory" factory-method="getPenInstance"/>

    <bean id="rule" factory-bean="suppliesFactory" factory-method="getRuleInstance"/>

public class SuppliesFactory {

    /**
     * 创建pen的工厂方法
     */
    public Pen getPenInstance() {
        return Pen.builder()
                .useWay("write")
                .name("hero")
                .writeStyle("hard")
                .build();
    }

    /**
     * 创建rule的工厂方法
     */
    public Rule getRuleInstance() {
        return Rule.builder()
                .name("bear")
                .useWay("clear")
                .build();
    }
}
```

如果是注解形式的话，直接在工厂方法上加入@bean注解

```
传统上构建对象的时候，如果这个对象依赖别的对象，那么在创建的时候需要先主动获取依赖的对象。
控制反转：在对象构建的时候，容器会主动提供需要依赖的对象，那么这个时候耦合性就变得很小。
依赖注入。通过外界来将对象的属性复制。提供一种赋值的方法即可。

该对象不查找其依赖项，并且不知道依赖项的位置或类。
```

**依赖注入存在两种主要的变体，基于构造函数的依赖注入和基于 Setter 的依赖注入。**

构造函数的依赖注入：

```java
// 基于构造函数的 依赖注入 是通过容器调用具有多个参数的构造函数来完成的，每个参数代表一个依赖项。

public class SimpleMovieLister {

  // the SimpleMovieLister has a dependency on a MovieFinder
  private MovieFinder movieFinder;

  // a constructor so that the Spring container can 'inject' a MovieFinder
  public SimpleMovieLister(MovieFinder movieFinder) {
      this.movieFinder = movieFinder;
  }

  // business logic that actually 'uses' the injected MovieFinder is omitted...
}
```

```xml
<!-- 在使用构造函数来进行依赖注入的时候，要明显区区别参数。
 	可以指定参数的下标（从0 开始）
	指定参数的类型
	Spring 3.0 之后可以指定参数的名称 在使用构造函数来进行依赖注入的时候，要明显区区别参数。-->

<bean id="exampleBean" class="examples.ExampleBean">
  <!--   参数类型 -->
	<constructor-arg type="int" value="7500000"/>
	<constructor-arg type="java.lang.String" value="42"/>
  
  <!-- 如果参数的类型没有继承关联时，可以指定对象   -->
 	<constructor-arg ref="baz"/>
  
  <!-- 指定下标：同类型   -->
  <constructor-arg index="1" value="42"/>
  
  <!-- 参数的名称   -->
	<constructor-arg name="ultimateanswer" value="42"/>

</bean>
```

**此时 构造方法可以在编译期间被执行，需要将这个注解加入到构造方法之上@ConstructorProperties **

基于 Setter 的依赖注入

```
基于 Setter 的 DI 是通过容器在 调用无参数构造函数或无参数静态工厂方法来实例化 bean 后 调用 bean 上的 setter 方法来完成的。
java 会默认提供无参构造函数

指定方式时通过      <property name="itemDao" ref="itemDao"/> 这种形式指定
```

**如果同时指定了构造函数、setter方法时，会优先使用Setter方法的**

```
ApplicationContext是用描述所有bean的配置元数据创建和初始化的。
```

**对于每个 bean，它的依赖项以属性、构造函数参数或静态工厂方法的参数的形式表示**

```
每个属性或构造函数参数都是要设置的值的实际定义，或者是对容器中另一个 bean 的引用。

作为值的每个属性或构造函数参数都从其指定格式转换为该属性或构造函数参数的实际类型。 默认情况下，Spring 可以将字符串格式提供的值转换为所有内置类型，例如 int、long、String、boolean . 字符串形式的value可以被转化成对应类型的值。
```

```
Spring 容器会在创建容器时验证每个 bean 的配置，包括验证 bean 引用属性是否引用有效 bean。

懒加载：
在实际创建 bean 之前不会设置 bean 属性本身。 创建容器时会创建单例范围并设置为预实例化（默认）的 Bean。
在实际应用的时候才会设置属性。
```

# 循环依赖

```
在使用构造函数进行依赖注入的时候，可能会导致循环依赖，A类通过构造函数依赖B类，B类通过构造函数依赖A类。并抛出并抛出 BeanCurrentlyInCreationException。
解决方法：使用Sett()方式注入bean
循环依赖导致一个对象没有完成之前被注入到另一个对象。
```

```
Spring 在容器加载时检测配置问题，比如对不存在的bean和循环依赖项的引用，有问题就报出，Spring尽可能晚地在实际创建bean时设置属性和解析依赖项。
```

在配置容器的时候发现问题，而不是在使用bean的时候发现问题。【预先加载的模式】

```
当您请求一个对象时，如果在创建该对象或其一个依赖项时出现问题，则正确加载的Spring容器可以生成异常。例如，由于缺少或无效的属性，bean会抛出异常。这可能会延迟一些配置问题的可见性【懒加载模式】
这就是ApplicationContext实现默认预实例化单例bean。在实际需要这些bean之前，您需要花费一些时间和内存来创建它们，在创建ApplicationContext时(而不是稍后)就会发现配置问题.
```

惰性加载bean在一定程度上解决了循环依赖的问题。

```
Spring 默认预先加载，而不是懒加载。
```

```
如果不存在循环依赖，当一个或多个协作 bean 被注入依赖 bean 时，每个协作 bean 在注入依赖 bean 之前都已完全配置。 协作bean已经完成了相关生命周期方法的调用
```

```java
静态工厂方法的参数通过 <constructor-arg/> 元素提供，与实际使用构造函数完全相同。
<bean id="exampleBean" class="examples.ExampleBean"
    factory-method="createInstance">
<constructor-arg ref="anotherExampleBean"/>
<constructor-arg ref="yetAnotherBean"/>
<constructor-arg value="1"/>
</bean>

public class ExampleBean {

  // a private constructor
  private ExampleBean(...) {
    ...
  }
  
  // a static factory method; the arguments to this method can be
  // considered the dependencies of the bean that is returned,
  // regardless of how those arguments are actually used.
  public static ExampleBean createInstance (
          AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

      ExampleBean eb = new ExampleBean (...);
      // some other operations...
      return eb;
  }
}
```

```
idref 元素只是一种将容器中另一个 bean 的 id（字符串值 - 而不是引用）传递给 <constructor-arg/> 或 <property/> 元素的防错方式。

<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
  <property name="targetName">
      <idref bean="theTargetBean" />
  </property>
</bean> 
更加安全。因为使用idref标记允许容器在部署时验证所引用的已命名bean是否实际存在。提前发现问题
在定义中的 AOP 拦截器的配置中。 在指定拦截器名称时使用 <idref/> 元素可防止您拼错拦截器 ID。
```

```xml
<!-- in the child (descendant) context -->
<bean id="accountService"  <-- bean name is the same as the parent bean -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/>  <!-- notice how we refer to the parent bean -->
    </property>
  <!-- insert other configuration and dependencies as required here -->
</bean>

```

```
内部 bean 定义不需要定义的 id 或名称； 容器忽略这些值。 它还忽略范围标志。 内部 bean 始终是匿名的，并且始终使用外部 bean 创建。
如果是静态内部类可以单独定义bean
```

```xml
<bean id="moreComplexObject" class="example.ComplexObject">
<!-- results in a setAdminEmails(java.util.Properties) call -->
<property name="adminEmails">
  <props>
      <prop key="administrator">administrator@example.org</prop>
      <prop key="support">support@example.org</prop>
      <prop key="development">development@example.org</prop>
  </props>
</property>
<!-- results in a setSomeList(java.util.List) call -->
<property name="someList">
  <list>
      <value>a list element followed by a reference</value>
    	<!--只有一个对象，是myDataSource对象->
      <ref bean="myDataSource" />
  </list>
</property>
<!-- results in a setSomeMap(java.util.Map) call -->
<property name="someMap">
  <map>
      <entry key="an entry" value="just some string"/>
      <entry key ="a ref" value-ref="myDataSource"/>
  </map>
</property>
<!-- results in a setSomeSet(java.util.Set) call -->
<property name="someSet">
  <set>
      <value>just some string</value>
      <ref bean="myDataSource" />
  </set>
</property>
</bean>
```

```xml
<beans>
<bean id="parent" abstract="true" class="example.ComplexObject">
  <property name="adminEmails">
      <props>
          <prop key="administrator">administrator@example.com</prop>
          <prop key="support">support@example.com</prop>
      </props>
  </property>
</bean>
<bean id="child" parent="parent">
  <property name="adminEmails">
      <!-- the merge is specified on the *child* collection definition -->
    	<!-- 子类中的集合会与父类中的集合合并--->
      <props merge="true">
          <prop key="sales">sales@example.com</prop>
          <prop key="support">support@example.co.uk</prop>
      </props>
  </property>
</bean>
<beans>
  
  这种合并行为同样适用于 <list/>、<map/> 和 <set/> 集合类型。 在 <list/> 元素的特定情况下，与 List 集合类型相关联的语义，即值的有序集合的概念，得到了维护； 父级的值在所有子级列表的值之前。但子类中如果与父类中有相同的key时，子类会覆盖父类的
```

```xml
<!---exampleBean.setEmail("") -->
<bean class="ExampleBean">
<property name="email" value=""/>
</bean>  


<!--exampleBean.setEmail(null).-->
<bean class="ExampleBean">
<property name="email"><null/></property>
</bean>
```

# 延迟加载

```
一个延迟初始化的 bean 告诉 IoC 容器在它第一次被请求时创建一个 bean 实例，而不是在启动时。

当 ApplicationContext 使用前面的配置时，ApplicationContext 启动时不会预先实例化名为 lazy 的 bean，而会预先实例化 not.lazy bean。但是，当延迟初始化 bean 是未延迟初始化的单例 bean 的依赖项时，ApplicationContext 在启动时创建延迟初始化 bean，因为它必须满足单例的依赖项。 延迟初始化的 bean 被注入到其他地方没有延迟初始化的单例 bean 中。

您还可以通过使用 <beans/> 元素上的 default-lazy-init 属性在容器级别控制延迟初始化；

<beans default-lazy-init="true">
  <!-- no beans will be pre-instantiated... -->
  <bean id="lazy" class="com.foo.ExpensiveToCreateBean" lazy-init="true"/>
</beans>
```

# 自动装配

```
Spring 容器可以自动装配协作 bean 之间的关系。 通过检查 ApplicationContext 的内容，您可以允许 Spring 自动为您的 bean 解析协作者（其他 bean）
```

自动装配的模式

```
NO:没有自动装配。 Bean 引用必须通过 ref 元素定义。 对于较大的部署，不建议更改默认设置，因为明确指定协作者可以提供更好的控制和清晰度
byName: Spring 查找与需要自动装配的属性同名的 bean。 例如，如果一个 bean 定义被设置为按名称自动装配，并且它包含一个 master 属性		（即它有一个 setMaster(..) 方法），Spring 会查找一个名为 master 的 bean 定义
byType:如果容器中只存在一个属性类型的 bean，则允许自动装配属性。 如果存在多个，则会引发致命异常,如果没有匹配的 bean，则什么也不会		发生；适用于setter模式的依赖注入
constructor:类似于 byType，但适用于构造函数参数。
```

**对应list和map 形式的对象，会将所有符合预期的bean注入进去**

```
局限：
显式设置的构造函数或者setter设置会覆盖自动装配
自动装配不如显式装配精确
容器内的多个 bean 定义可能与要自动装配的 setter 方法或构造函数参数指定的类型相匹配。 然而，对于期望单个值的依赖项，这种歧义不会被任意解决。 如果没有唯一的 bean 定义可用，则抛出异常。
```

# 方法注入

```
一个单例的bean 依赖一个多实例的bean，希望每次获取不同的bean.

假设单例 bean A 需要使用非单例（原型）bean B，可能在 A 上的每次方法调用上。容器只创建单例 bean A 一次，因此只有一次设置属性的机会。 容器无法在每次需要时为 bean A 提供 bean B 的新实例。
```

方法一： 单例类实现ApplicationContextAware

```java
// 通过感知容器来获取容器内对象。此方法不可取，与业务代码耦合啦
public class CommandManager implements ApplicationContextAware {

 private ApplicationContext applicationContext;

 public Object process(Map commandState) {
    // grab a new instance of the appropriate Command
    Command command = createCommand();
    // set the state on the (hopefully brand new) Command instance
    command.setState(commandState);
    return command.execute();
 }

 protected Command createCommand() {
    // notice the Spring API dependency!
   // 从容器中获取多实例对象
    return this.applicationContext.getBean("command", Command.class);
 }

 public void setApplicationContext(ApplicationContext applicationContext)
                                                                  throws BeansException {
    this.applicationContext = applicationContext;
 }
}
```

方法二、方法查找

```
查找方法注入是容器覆盖容器管理 bean 上的方法的能力，以返回容器中另一个命名 bean 的查找结果。 查找通常涉及原型 bean
Spring Framework 通过使用来自 CGLIB 库的字节码动态生成覆盖该方法的子类 来实现此方法注入。
要使这种动态子类化工作，Spring 容器子类化的类不能是 final，要覆盖的方法也不能是 final。

方法的格式：<public|protected> [abstract] <return-type> theMethodName(no-arguments);
如果该方法是抽象的，则动态生成的子类将实现该方法。 否则，动态生成的子类会覆盖原类中定义的具体方法

```

```java
package fiona.apple;

// no more Spring imports! 

public abstract class CommandManager {

 public Object process(Object commandState) {
    // grab a new instance of the appropriate Command interface
    Command command = createCommand();
    // set the state on the (hopefully brand new) Command instance
    command.setState(commandState);
    return command.execute();
 }

  // okay... but where is the implementation of this method?
 @Lookup
 protected abstract Command createCommand();
}
```

```xml
package fiona.apple;

// no more Spring imports! 

public abstract class CommandManager {

 public Object process(Object commandState) {
    // grab a new instance of the appropriate Command interface
    Command command = createCommand();
    // set the state on the (hopefully brand new) Command instance
    command.setState(commandState);
    return command.execute();
 }

  // okay... but where is the implementation of this method?
 protected abstract Command createCommand();
}

<bean id="command" class="fiona.apple.AsyncCommand" scope="prototype">
<!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
  <!-- 注入属性-->
<lookup-method name="createCommand" bean="command"/>
</bean>
```

# bean 作用域

```
您不仅可以控制要插入到从特定 bean 定义创建的对象中的各种依赖项和配置值，还可以控制从特定 bean 定义创建的对象的范围。
```

```
singleton:（默认）将单个 bean 定义范围限定为每个 Spring IoC 容器的单个对象实例。
prototype:将单个 bean 定义范围限定为任意数量的对象实例。
request:将单个 bean 定义范围限定为单个 HTTP 请求的生命周期,仅对web容器有用
session:将单个 bean 定义范围限定为 HTTP 会话的生命周期。仅对web容器有用
global session：将单个 bean 定义范围限定为全局 HTTP 会话的生命周期。
```

```
当您定义一个 bean 定义并且它的作用域是一个单例时，Spring IoC 容器会创建该 bean 定义定义的对象的一个​​实例。 该单个实例存储在此类单例 bean 的缓存中，并且对该命名 bean 的所有后续请求和引用都返回缓存对象。
```

```
bean 部署的非单一原型范围导致每次对特定 bean 发出请求时都会创建一个新 bean 实例。
对所有有状态 bean 使用原型作用域，对无状态 bean 使用单例作用域。
```

Spring 不管理原型 bean 的完整生命周期：容器实例化、配置和组装原型对象，并将其交给客户端.尽管在所有对象上调用初始化生命周期回调方法而不考虑范围，但在原型的情况下，不会调用配置的销毁生命周期回调。 客户端代码必须清理原型范围内的对象并释放原型 bean 持有的昂贵资源。

# 生命周期

```
要与容器对 bean 生命周期的管理进行交互，您可以实现 Spring InitializingBean 和 DisposableBean 接口。 容器为前者调用 afterPropertiesSet()，为后者调用 destroy() 以允许 bean 在初始化和销毁 bean 时执行某些操作。
```

@PostConstruct 和 @PreDestroy 注释通常被认为是在现代 Spring 应用程序中接收生命周期回调的最佳实践

```java 
在内部，Spring 框架使用 BeanPostProcessor 实现来处理它可以找到的任何回调接口并调用适当的方法。
```

```
除了初始化和销毁​​回调之外，Spring 管理的对象还可以实现 Lifecycle 接口，以便这些对象可以参与由容器自身生命周期驱动的启动和关闭过程。
```

## Initialization callbacks

```
在pojo类中 init方法上使用@PostConstruct 注解，调用时机在设置bean的基本属性之后
```

```java
public class ExampleBean {

  @PostConstruct
  public void init() {
      // do some initialization work
  }
}

public class AnotherExampleBean implements InitializingBean {

  public void afterPropertiesSet() {
      // do some initialization work
  }
}
```

## Destruction callbacks

```
实现disposable bean接口允许bean在容器被销毁时获得回调。  调用时机：在bean销毁之前调用
```

```java
public class ExampleBean {

@PreDestroy
  public void cleanup() {
      // do some destruction work (like releasing pooled connections)
  }
}

public class AnotherExampleBean implements DisposableBean {

  public void destroy() {
      // do some destruction work (like releasing pooled connections)
  }
}
```

```
顶级 <beans/> 元素属性上的 default-init-method 属性的存在导致 Spring IoC 容器将 bean 上称为 init 的方法识别为初始化方法回调。
您可以通过使用顶级 <beans/> 元素上的 default-destroy-method 属性类似地（即在 XML 中）配置销毁方法回调。
现有 bean 类已经具有命名与约定不同的回调方法，您可以通过使用 <bean/> 的 init-method 和 destroy-method 属性指定（在 XML 中，即）方法名称来覆盖默认值 本身。
```

初始化执行顺序

```
@PostConstruct
afterPropertiesSet() as defined by the InitializingBean callback interface
A custom configured init() method  指定的初始化方法 @Bean(initMethod = "init",destroyMethod = "destory")
```

销毁执行顺序

```
@PreDestroy
destroy() as defined by the DisposableBean callback interface
destroy()
```

Lifecycle 接口为任何具有自己生命周期要求的对象定义了基本方法

```java
public interface LifecycleProcessor extends Lifecycle {

  void onRefresh();

  void onClose();

}
```

 ```
 LifecycleProcessor 本身是 Lifecycle 接口的扩展。 它还添加了另外两种方法来对正在刷新和关闭的上下文做出反应。
 
 一个实现 SmartLifecycle 并且其 getPhase() 方法返回 Integer.MIN_VALUE 的对象将是最先启动和最后一个停止的对象。
 getPhase() 控制着启动和关闭的优先权。值越大，最优先启动，越最后关闭
 ```

# 优雅关闭IOC

```java
你向 JVM 注册了一个关闭钩子。 这样做可确保正常关闭并在单例 bean 上调用相关的 destroy 方法，以便释放所有资源

public static void main(final String[] args) throws Exception {
      AbstractApplicationContext ctx
          = new ClassPathXmlApplicationContext(new String []{"beans.xml"});

      // add a shutdown hook for the above context... 
      ctx.registerShutdownHook();

      // app runs here...

      // main method exits, hook is called prior to the app shutting down...
  }
```

### `ApplicationContextAware`

用于感知配置信息

```
当 ApplicationContext 创建一个实现 org.springframework.context.ApplicationContextAware 接口的类时，该类提供了对该 ApplicationContext 的引用。
能够感知容器。

public interface ApplicationContextAware {

  void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}

具体就是通过这个接口来获取容器。可以使用自动装配机制，来通过@Autowired 注入一个容器。
```

```
当 ApplicationContext 创建一个实现 org.springframework.beans.factory.BeanNameAware 接口的类时，该类被提供了对在其关联对象定义中定义的名称的引用。

public interface BeanNameAware {

  void setBeanName(string name) throws BeansException;
}
```

# Bean 定义信息的继承

```
bean定义可以包含大量配置信息，包括构造函数参数、属性值和特定于容器的信息，比如初始化方法、静态工厂方法名，等等。子bean定义从父定义继承配置数据。
```

```xml
<!--bean 继承定义-->
<bean id="inheritedTestBean" abstract="true"
    class="org.springframework.beans.TestBean">
  <property name="name" value="parent"/>
  <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
      class="org.springframework.beans.DerivedTestBean"
      parent="inheritedTestBean" init-method="initialize">

  <property name="name" value="override"/>
  <!-- the age property value of 1 will be inherited from  parent -->

</bean>
```

容器的内部 preInstantiateSingletons() 方法忽略定义为抽象的 bean 定义。

# BeanPostProcessor

```
BeanPostProcessor 接口定义了回调方法，您可以实现这些方法来提供您自己的（或覆盖容器的默认）实例化逻辑、依赖解析逻辑等。
你可以配置多个 BeanPostProcessor 实例，你可以通过设置 order 属性来控制这些 BeanPostProcessor 的执行顺序。 只有在 BeanPostProcessor 实现了 Ordered 接口时才能设置此属性；
```

**Spring IoC 容器实例化一个 bean 实例，然后 BeanPostProcessors 做它们的工作。**

**BeanPostProcessors 是针对每个容器的。 这仅在您使用容器层次结构时才相关。**

```
org.springframework.beans.factory.config.BeanPostProcessor 接口正好包含两个回调方法。 
对于容器创建的每个 bean 实例，后处理器在容器初始化方法（例如 InitializingBean 的 afterPropertiesSet() 和任何 声明的 init 方法）以及在任何 bean 初始化回调之后调用。

bean 后处理器通常会检查回调接口或可能使用代理包装 bean。 一些 Spring AOP 基础设施类被实现为 bean 后处理器，以提供代理包装逻辑。
```

ApplicationContext将这些实现BeanPostProcessor接口的配置元数据中定义的任何bean注册为后处理器，以便在稍后创建bean时调用它们

```
虽然推荐的 BeanPostProcessor 注册方法是通过 ApplicationContext 自动检测（如上所述），但也可以使用 addBeanPostProcessor 方法以编程方式针对 ConfigurableBeanFactory 注册它们。 这在需要在注册之前评估条件逻辑时非常有用，或者甚至用于跨层次结构中的上下文复制 bean 后处理器。 
但是请注意，以编程方式添加的 BeanPostProcessors 不遵守 Ordered 接口。 在这里，注册顺序决定了执行的顺序。 另请注意，以编程方式注册的 BeanPostProcessor 始终在通过自动检测注册的 BeanPostProcessor 之前处理，无论任何显式排序如何。
```

```java
package scripting;

import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.beans.BeansException;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

  // simply return the instantiated bean as-is
  public Object postProcessBeforeInitialization(Object bean, String beanName)
                                                                     throws BeansException {
      return bean; // we could potentially return any object reference here...
  }

  // 它在容器创建时调用每个 bean 的 toString() 方法并将结果字符串打印到系统控制台
  public Object postProcessAfterInitialization(Object bean, String beanName)
                                                                     throws BeansException {
      System.out.println("Bean '" + beanName + "' created : " + bean.toString());
      return bean;
  }
}
```

**将回调接口或注解与自定义 BeanPostProcessor 实现结合使用是扩展 Spring IoC 容器的常用方法。 通过对后置处理器进行配置，在初始化生命周期之后，对注解进行处理**

```
RequiredAnnotationBeanPostProcessor，它确保用(任意)注释标记的bean上的JavaBean属性实际上(配置为)依赖注入一个值。
```

# BeanFactoryPostProcessor

```
与BeanPostProcessor一样，使用BeanFactoryPostProcessor自定义配置元数据。对生成bean的工厂进行后置处理

BeanFactoryPostProcessors操作bean配置元数据;也就是说，Spring IoC容器允许BeanFactoryPostProcessors读取配置元数据，并在容器实例化BeanFactoryPostProcessors之外的任何bean之前可能更改它。

您可以配置多个BeanFactoryPostProcessors，并且可以通过设置order属性来控制这些BeanFactoryPostProcessors的执行顺序。但是，只有在BeanFactoryPostProcessor实现了Ordered接口时，才能设置此属性。
```

不建议使用BeanFactoryPostProcessor 来修改bean

**您在<beans />元素的声明中将default-lazy-init属性设置为true, Bean(Factory)PostProcessor也将被立即实例化。**

```
PropertyPlaceholderConfigurer:用于填充对象属性，属性可以从配置文件,xml中获取
使用PropertyPlaceholderConfigurer将bean定义中的属性值外部化到使用标准Java Properties格式的单独文件中,通过$ 再引入  
其中定义了一个带有占位符值的DataSource。该示例显示了从外部properties文件配置的属性。在运行时，PropertyPlaceholderConfigurer被应用于元数据，它将替换数据源的一些属性。要替换的值被指定为表单${property-name}的占位符，该表单遵循Ant / log4j / JSP EL风格。


PropertyPlaceholderConfigurer不仅在您指定的properties文件中寻找属性。默认情况下，如果在指定的属性文件中找不到属性，它也会检查Java System属性。
```

```
PropertyOverrideConfigurer 是另一个 bean 工厂后处理器，类似于 PropertyPlaceholderConfigurer，但与后者不同的是，原始定义可以具有 bean 属性的默认值或根本没有值。 如果覆盖的属性文件没有特定 bean 属性的条目，则使用默认上下文定义。

 如果多个 PropertyOverrideConfigurer 实例为同一个 bean 属性定义了不同的值，由于覆盖机制，最后一个获胜。
 
例如：
dataSource.driverClassName=com.mysql.jdbc.Driver
dataSource.url=jdbc:mysql:mydb
此示例文件可与包含名为 dataSource 的 bean 的容器定义一起使用，该 bean 具有 driver 和 url 属性。
```

# 使用 FactoryBean 自定义实例化逻辑 

```java
FactoryBean 接口是 Spring IoC 容器实例化逻辑的可插入点。 如果您有复杂的初始化代码，可以用 Java 更好地表达，而不是（可能）冗长的 XML，您可以创建自己的 FactoryBean，在该类中编写复杂的初始化，然后将您的自定义 FactoryBean 插入到容器中。


Object getObject()：返回此工厂创建的对象的实例。 实例可能会被共享，这取决于这个工厂是返回单例还是原型。

boolean isSingleton()：如果此 FactoryBean 返回单例，则返回 true，否则返回 false。

Class  getObjectType()：返回 getObject() 方法返回的对象类型，如果类型事先未知，则返回 null。
  
  
当您需要向容器询问实际的 FactoryBean 实例本身而不是它生成的 bean 时，在调用 ApplicationContext 的 getBean() 方法时，在 bean 的 id 前面加上与符号 (&)。 否则会返回工厂创建的对象
```

**注释注入在XML注入之前执行，因此对于通过两种方法连接的属性，后一种配置将覆盖前者。**

```xml
<context:annotation-config/>只在定义它的应用程序上下文中查找bean上的注释。


<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xmlns:context="http://www.springframework.org/schema/context"
     xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans.xsd
         http://www.springframework.org/schema/context
         http://www.springframework.org/schema/context/spring-context.xsd">

   <context:annotation-config/>

</beans>
```

@`Required`

```java
public class SimpleMovieLister {

  private MovieFinder movieFinder;

  @Required
  public void setMovieFinder(MovieFinder movieFinder) {
      this.movieFinder = movieFinder;
  }

  // ...
}

必须在配置时通过bean定义中的显式属性值或通过自动装配来填充受影响的bean属性。如果未填充受影响的bean属性，容器将抛出异常;这允许紧急和显式的失败，避免以后出现nullpointerexception或类似的错误。
```

**@Autowired. 通过将注解添加到需要该类型数组的字段或方法，还可以从 ApplicationContext 提供特定类型的所有 bean：**

```java
@Autowired
private MovieCatalog[] movieCatalogs;

//只要预期的键类型是字符串，即使是类型化的 Map 也可以自动装配。 Map 值将包含预期类型的​​所有 bean，键将包含相应的 bean 名称：

@Autowired
  public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
      this.movieCatalogs = movieCatalogs;
  }


// 如果属性为空的时候，不抛出异常
@Autowired(required=false)
  public void setMovieFinder(MovieFinder movieFinder) {
      this.movieFinder = movieFinder;
  }
```

````
推荐使用@Autowired的required属性而不是@Required注释。required属性指示该属性对于自动连接目的不是必需的，如果不能自动连接该属性，则忽略该属性。另一方面，@Required更强，因为它强制使用容器支持的任何方式设置的属性。如果没有注入值，则会引发相应的异常

您还可以为那些众所周知的可解析依赖项使用@Autowired: BeanFactory、ApplicationContext、Environment、ResourceLoader、ApplicationEventPublisher和MessageSource。

@Autowired
  private ApplicationContext context;
````

**@Autowired、@Inject、@Resource和@Value注解是由Spring BeanPostProcessor实现处理的  这意味着你不能在自己的BeanPostProcessor或BeanFactoryPostProcessor类型(如果有的话)中应用这些注解。这些类型必须通过XML或使用Spring @Bean方法显式地“连接起来”。**

```java
按类型自动装配可能会导致多个候选人，因此通常有必要对选择过程有更多的控制。
@Qualifier 能够指定注入bean 的名称

    @Component("fooFormatter")
    public class FooFormatter implements Formatter {
        public String format() {
            return "foo";
        }
    }
    @Component("barFormatter")
    public class BarFormatter implements Formatter {
        public String format() {
            return "bar";
        }
    }
    @Component
    public class FooService {
        @Autowired
        private Formatter formatter;
       //Spring 框架将抛出 NoUniqueBeanDefinitionException。这是因为 Spring 不知道要注入哪个 bean。
        //todo 
    }

        // 根据具体的bean 名称来注入对象
				@Autowired
        @Qualifier("fooFormatter")
        private Formatter formatter;

// 设置bean名称的方法：
1、@Component或者@Bean注解中声明的value属性以确定名称
2、在实现类上使用 @Qualifier 注释
  @Component
  @Qualifier("barFormatter")
  public class BarFormatter implements Formatter

    
    @Qualifier 可以设置在集合上，表示收集所有符合集合名称的对象，在bean定义的时候可以为多个对象设置重复的值
```

``` java
@Primary 注释关联的 bean 表示优先注入
@Bean
    @Primary
    public Employee johnEmployee() {
        return new Employee("john");
    }

当我们想要指定默认情况下应该注入特定类型的 bean 时
@Qualifier 和 @Primary 注释都存在，那么 @Qualifier 注释将具有优先权。
```

```
如果您打算通过名称表示注释驱动的注入，那么不要主要使用@Autowired，使用 @Resource注释，它在语义上定义为通过其惟一名称标识特定的目标组件，声明的类型与匹配过程无关。

不能通过@Autowired注入本身定义为集合或映射类型的bean

@Autowired应用于字段、构造函数和多参数方法，允许在参数级别通过限定符注释缩小范围。相比之下，@Resource只支持具有单个参数的字段和bean属性设置方法。
```

# 自定义注解

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Genre {

  String value();
}

// 使用
public class MovieRecommender {

  @Autowired
  @Genre("Action")
  private MovieCatalog actionCatalog;

  private MovieCatalog comedyCatalog;

  @Autowired
  public void setComedyCatalog(@Genre("Comedy") MovieCatalog comedyCatalog) {
      this.comedyCatalog = comedyCatalog;
  }
  // ...
}
```

*多个bean符合自动装配候选时，“主”候选的确定是相同的:如果候选中的一个bean定义的主属性设置为true，那么它将被选中。*

# `@Resource`

```java
public class SimpleMovieLister {

  private MovieFinder movieFinder;

  @Resource(name="myMovieFinder")
  public void setMovieFinder(MovieFinder movieFinder) {
      this.movieFinder = movieFinder;
  }
}
如果未显式指定名称，则默认名称派生自字段名或setter方法。对于字段，它接受字段名;对于setter方法，它接受bean属性名。
```

```
与注释一起提供的名称由CommonAnnotationBeanPostProcessor感知的ApplicationContext解析为bean名称。如果显式配置Spring的SimpleJndiBeanFactory，则可以通过JNDI解析这些名称。
```

```java
CommonAnnotationBeanPostProcessor不仅可以识别@Resource注释，还可以识别JSR-250生命周期注释。
如果CommonAnnotationBeanPostProcessor是在Spring ApplicationContext中注册的，那么携带这些注释之一的方法将在生命周期中的同一点被调用，与相应的Spring生命周期接口方法或显式声明的回调方法相同。
  
public class CachingMovieLister {

  @PostConstruct
  public void populateMovieCache() {
      // populates the movie cache upon initialization...
  }

  @PreDestroy
  public void clearMovieCache() {
      // clears the movie cache upon destruction...
  }
}

```





如果类比较纯粹，没有@Transactional ,没有被代理的话，容器中的对象是原始对象，否则就是代理对象
