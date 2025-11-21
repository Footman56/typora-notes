两个注解能自动装配对象。

注入方式

```java
    // 构造方法注入
    @Autowired
    public Service(Service service) {
        this.service = service;
    }

    // 成员变量注入
    @Autowired
    private Service service;
 
    // 方法参数注入
    @Autowired
    public void setService(Service service) {
        this.service = service;
    }
```



# 区别：

## 参数不同

@Autowired：需要一个required 参数，表示bean 是否必须存在，不能指定姓名

```java
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired {

	/**
	 * Declares whether the annotated dependency is required.
	 * <p>Defaults to {@code true}.
	 */
	boolean required() default true;
}
```

@Resource： name 指定bean名称、type 指定be an类型

```java
@Target({TYPE, FIELD, METHOD})
@Retention(RUNTIME)
public @interface Resource {
    /**
    * bean 名称
     */
    String name() default "";

    /**
     * The name of the resource that the reference points to. It can
     * link to any compatible resource using the global JNDI names.
     *
     * @since Common Annotations 1.1
     */

    String lookup() default "";

    /**
     * 引用类型：
     * 对于字段注释，默认值为字段的类型。
     * 对于方法注解，缺省值为 JavaBeans 属性的类型。
     * 对于类注释，没有默认值，必须指定。 
     */
    Class<?> type() default java.lang.Object.class;

    /**
     *
     */
    enum AuthenticationType {
            CONTAINER,
            APPLICATION
    }

    /**
     * 身份验证类型
     */
    AuthenticationType authenticationType() default AuthenticationType.CONTAINER;

    /**
     *  组件是否可以与其他组件共享
     */
    boolean shareable() default true;

    /**

     */
    String mappedName() default "";

    /**
     * 描述
     */
    String description() default "";
}
```

## 出处不同

@Autowired 是Spring 提供的

@Resource 是JDK 提供的

## 作用范围不同

@Autowired  构造方法、成员变量、方法参数、方法及注解上，

@Resource能用在类、成员变量、方法参数上

## 装配方式的默认值不同

@Autowired默认按type自动装配

@Resource默认按name自动装配。@Resource注解可以自定义选择装配方式，如果指定name，则按name自动装配。如果指定type，则按type自动装配。如果没有显式声明名称则按照变量名称或者方法中对应的参数名称进行注入。

## 装载方式不同

@Autowired ： 先根据byType 去查找，如果查到唯一一个就是所得。

如果查找多个bean检查是否有@Qualifier,如果有根据@Qualifier参数查找，没有找到抛异常

如果没有@Qualifier ，根据名称查找Bean,如果没有 再判断requird 是否 为 true ，为true抛异常,否则返回null

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202306281140997.png" alt="image-20230628114014946" style="zoom:50%;" />

从**Spring Framework 4.3**开始，`@Autowired`如果目标Bean只定义一个构造函数，则不再需要在该构造函数上添加`@Autowired`注解。如果目标Bean有几个构造函数可用，并且没有主/默认构造函数，则必须至少有一个构造函数被`@Autowired`标记，以指示Spring IoC容器使用哪个构造函数。







@Resource：

先根据name匹配,如果配置到，检查type 是否与bean 类型一致，不一致抛异常。一致的话再检查是否有@Qualifier 约束，满足约束条件，再检查是否有多个符合条件的，如果有多个符合条件的就抛异常，否则找到bean;

如果根据name没有匹配到，再根据type 查找，如果type不存在就抛异常。否则就再检查是否有@Qualifier 约束，满足约束条件，再检查是否有多个符合条件的，如果有多个符合条件的就抛异常，否则找到bean;

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202306281133211.png" alt="image-20230628113330151" style="zoom:50%;" />