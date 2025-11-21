Condition 接口 用于判断类是否可以加入到容器中，【通过match 方法来判断】

# 功能

+ 灵活的判断是否需要加载bean

+ 通过 `@Conditional` 注解，`Condition` 接口的实现类可以用于配置类或者配置类中的方法上，从而条件化地应用或排除某个配置。



# 实现

```java
 /**
     * 确定条件是否匹配。
     * @param context 条件上下文
     * @param metadata 被检查的 org.springframework.core.type.AnnotationMetadata类或org.springframework.core.type.MethodMetadata方法的元数据
     * @return true 如果条件匹配且组件可以注册，或 false 以否决带有注解组件的注册
     */
    boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
```



# 子类

+ ProfileCondition ：通过判断运行时的profile信息，决定是否注册组件。例如，可以根据激活的profile选择性地注册特定的Bean。
+ OnBeanCondition ： 根据容器中是否存在某个特定类型的Bean来进行条件判断，决定是否注册组件。这个条件允许根据容器中的Bean的存在与否来决定是否创建某个Bean。
+ OnClassCondition ： 判断类路径中是否存在某个特定的类，决定是否注册组件。通过检查类路径，可以根据类的存在与否来动态控制组件的注册。
+ OnPropertyCondition ： 根据配置文件中的属性值来进行条件判断，决定是否注册组件。可以根据配置文件中的属性来灵活地配置组件的注册。
+ ResourceCondition ： 判断类路径中是否存在指定资源文件，决定是否注册组件。类似于`OnClassCondition`，但是可以判断任意资源文件的存在与否。



以OnPropertyCondition 举例：

1. 首先是@ConditionalOnProperty注解的功能，用于匹配properties 属性，value 字段
2. @ConditionalOnProperty  包含@Conditional，@Conditional配置的自定义Condition 接口为 OnPropertyCondition 类
3. OnPropertyCondition的实现 ： 获取类或者方法上@ConditionalOnProperty注解配置的参数，与环节参数去匹配

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ ElementType.TYPE, ElementType.METHOD })
@Documented
@Conditional(OnPropertyCondition.class)
public @interface ConditionalOnProperty {

	/**
	 * Alias for {@link #name()}.
	 * @return the names
	 */
	String[] value() default {};

	/**
	 * A prefix that should be applied to each property. The prefix automatically ends
	 * with a dot if not specified. A valid prefix is defined by one or more words
	 * separated with dots (e.g. {@code "acme.system.feature"}).
	 * @return the prefix
	 */
	String prefix() default "";

	/**
	 * The name of the properties to test. If a prefix has been defined, it is applied to
	 * compute the full key of each property. For instance if the prefix is
	 * {@code app.config} and one value is {@code my-value}, the full key would be
	 * {@code app.config.my-value}
	 * <p>
	 * Use the dashed notation to specify each property, that is all lower case with a "-"
	 * to separate words (e.g. {@code my-long-property}).
	 * @return the names
	 */
	String[] name() default {};

	/**
	 * The string representation of the expected value for the properties. If not
	 * specified, the property must <strong>not</strong> be equal to {@code false}.
	 * @return the expected value
	 */
	String havingValue() default "";

	/**
	 * Specify if the condition should match if the property is not set. Defaults to
	 * {@code false}.
	 * @return if should match if the property is missing
	 */
	boolean matchIfMissing() default false;

}
```



```java
class OnPropertyCondition extends SpringBootCondition {

	@Override
	public ConditionOutcome getMatchOutcome(ConditionContext context, AnnotatedTypeMetadata metadata) {
    // 获取类或者方法上的ConditionalOnProperty注解参数
		List<AnnotationAttributes> allAnnotationAttributes = annotationAttributesFromMultiValueMap(
				metadata.getAllAnnotationAttributes(ConditionalOnProperty.class.getName()));
		List<ConditionMessage> noMatch = new ArrayList<>();
		List<ConditionMessage> match = new ArrayList<>();
    // 遍历所有参数
		for (AnnotationAttributes annotationAttributes : allAnnotationAttributes) {
      // 判断是否有环节参数
			ConditionOutcome outcome = determineOutcome(annotationAttributes, context.getEnvironment());
			(outcome.isMatch() ? match : noMatch).add(outcome.getConditionMessage());
		}
		if (!noMatch.isEmpty()) {
			return ConditionOutcome.noMatch(ConditionMessage.of(noMatch));
		}
		return ConditionOutcome.match(ConditionMessage.of(match));
	}
}
```



**这种实现类比于 A调用B ,而B的实现是处理A逻辑的**



# 使用

```java
public static void main(String[] args) throws IOException {
        // 创建资源解析器，用于获取匹配指定模式的资源
        PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();

        // 创建MetadataReader工厂，用于读取类的元数据信息
        SimpleMetadataReaderFactory metadataReaderFactory = new SimpleMetadataReaderFactory();

        // 获取指定模式下的所有资源
        Resource[] resources = resolver.getResources("classpath*:com/xcs/spring/bean/**/*.class");

        // 创建自定义条件类的实例，用于条件匹配
        Condition condition = new MyOnClassCondition("com.xcs.spring.ConditionDemo");

        // 遍历每个资源，判断是否满足自定义条件
        for (Resource resource : resources) {
            // 获取资源对应的元数据读取器
            MetadataReader metadataReader = metadataReaderFactory.getMetadataReader(resource);

            // 判断资源是否满足自定义条件
            if (condition.matches(null, metadataReader.getAnnotationMetadata())) {
                System.out.println(resource.getFile().getName() + "满足条件");
            } else {
                System.out.println(resource.getFile().getName() + "不满足条件");
            }
        }
    }
```



# 注意

+ 当条件判断出现错误时，例如 `matches` 方法中抛出异常，可能会导致注册组件的失败。我们需要注意条件逻辑中的异常处理，以确保不会影响到整个应用的启动。

+ 如果应用中有大量的条件，可能会影响启动性能

+ 条件是用于决定是否注册组件的机制，而配置则是用于配置组件的属性和行为。

+ 条件的匹配是在什么时候发生的。条件是在 bean 的定义注册之前立即进行检查的，因此可以在 Spring 应用上下文初始化之前进行条件匹配。

  



