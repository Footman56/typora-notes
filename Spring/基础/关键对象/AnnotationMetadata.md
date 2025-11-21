# AnnotationMetadata

重点用于获取 **类上的注解，类中方法的注解**

用于访问和分析与类、方法、字段等元素相关的注解信息。它提供了一种方便的方式，让我们可以在运行时动态地获取和操作类上的注解信息。

+ `hasAnnotation(String annotationName)` 方法来检查是否存在指定名称的注解在类上。这对于条件化的配置非常有用，可以根据类上的注解来控制不同的行为。
+  `getAnnotationAttributes(String annotationName)` 方法，我们可以获取指定注解的属性值
+ `getAnnotations()` 方法返回类上所有的注解，以 `AnnotationAttributes` 对象的列表形式
+ 通过 `getMethodMetadata()`、`getFieldMetadata()` 和 `getClassMetadata()` 方法，我们可以获取类中方法、字段和类本身的元数据信息

**具体有两个子类 `StandardAnnotationMetadata`,`SimpleAnnotationMetadata`**

## 

这个实现依赖于标准的 Java 反射机制，它使用 Java 的 `java.lang.Class` 对象来分析和访问类的注解信息。通过这个实现，我们可以轻松地检查类上的注解、获取注解属性值以及执行其他与注解相关的操作

```java
// 获取 AnnotationMetadata-通过反射方式获取
        AnnotationMetadata annotationMetadata = AnnotationMetadata.introspect(MyBean.class);

        System.out.println("AnnotationMetadata impl class is " + annotationMetadata.getClass());

        // 检查 MyBean 类是否被 @Component 注解标记
        String name = Component.class.getName();
        System.out.println("name = " + name);
        boolean isComponent = annotationMetadata.hasAnnotation(name);
        System.out.println("MyBean is a @Component: " + isComponent);
        // 获取 MyBean 类上的注解属性
        if (isComponent) {
            Map<String, Object> annotationAttributes = annotationMetadata.getAnnotationAttributes(name);
            System.out.println("@Component value is " + annotationAttributes.get("value"));
        }
        // 获取含有指定注解的方法
        Set<MethodMetadata> annotatedMethods = annotationMetadata.getAnnotatedMethods(Bean.class.getName());
        if (!CollectionUtils.isEmpty(annotatedMethods)){
            annotatedMethods.forEach(methodMetadata -> {
                String methodName = methodMetadata.getMethodName();
                System.out.println("methodName = " + methodName);
            });
        }
        // 此方法获取不到字段上的注解
```



## SimpleAnnotationMetadata

是一个基于 ASM（字节码操作库）的实现，用于分析和访问类的注解信息。相比于 `StandardAnnotationMetadata`，它通常更轻量，不依赖于标准 Java 反射机制，而是直接解析类的字节码。

```java
// 创建 MetadataReaderFactory-ASM方式
        SimpleMetadataReaderFactory readerFactory = new SimpleMetadataReaderFactory();
        // 获取 MetadataReader
        MetadataReader metadataReader = readerFactory.getMetadataReader("com.xcs.spring.bean.MyBean");
        // 获取 AnnotationMetadata
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();

        System.out.println("AnnotationMetadata impl class is " + annotationMetadata.getClass());

        // 检查 MyBean 类是否被 @Component 注解标记
        boolean isComponent = annotationMetadata.hasAnnotation(Component.class.getName());
        System.out.println("MyBean is a @Component: " + isComponent);

        // 获取 MyBean 类上的注解属性
        if (isComponent) {
            Map<String, Object> annotationAttributes = annotationMetadata.getAnnotationAttributes(Component.class.getName());
            System.out.println("@Component value is " + annotationAttributes.get("value"));
        }
```



# MetadataReader

允许应用程序获取有关类的元数据信息，包括类的名称、访问修饰符、接口、超类、注解等等

```java
public interface MetadataReader {

	/**
	 * Return the resource reference for the class file.
	 */
	Resource getResource();
	/**
	 * 用于标识类、父类的信息
	 */
	ClassMetadata getClassMetadata();
	/**
	* 用于标识类上、方法上的注解元信息
	 */
	AnnotationMetadata getAnnotationMetadata();

}
```

```java
// 创建 MetadataReaderFactory
        SimpleMetadataReaderFactory readerFactory = new SimpleMetadataReaderFactory();
        // 获取 MetadataReader，通常由 Spring 容器自动创建
        MetadataReader metadataReader = readerFactory.getMetadataReader("com.xcs.spring.bean.MyBean");

        // 获取类的基本信息
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        System.out.println("Class Name = " + classMetadata.getClassName());
        System.out.println("Class IsInterface = " + classMetadata.isInterface());
        System.out.println("Class IsAnnotation = " + classMetadata.isAnnotation());
        System.out.println("Class IsAbstract = " + classMetadata.isAbstract());
        System.out.println("Class IsConcrete = " + classMetadata.isConcrete());
        System.out.println("Class IsFinal = " + classMetadata.isFinal());
        System.out.println("Class IsIndependent = " + classMetadata.isIndependent());
        System.out.println("Class HasEnclosingClass = " + classMetadata.hasEnclosingClass());
        System.out.println("Class EnclosingClassName = " + classMetadata.getEnclosingClassName());
        System.out.println("Class HasSuperClass = " + classMetadata.hasSuperClass());
        System.out.println("Class SuperClassName = " + classMetadata.getSuperClassName());
        System.out.println("Class InterfaceNames = " + Arrays.toString(classMetadata.getInterfaceNames()));
        System.out.println("Class MemberClassNames = " + Arrays.toString(classMetadata.getMemberClassNames()));
        System.out.println("Class Annotations: " +  metadataReader.getAnnotationMetadata().getAnnotationTypes());

        System.out.println();

        // 获取方法上的注解信息
        for (MethodMetadata methodMetadata : metadataReader.getAnnotationMetadata().getAnnotatedMethods("com.xcs.spring.annotation.MyAnnotation")) {
            System.out.println("Method Name: " + methodMetadata.getMethodName());
            System.out.println("Method DeclaringClassName: " + methodMetadata.getDeclaringClassName());
            System.out.println("Method ReturnTypeName: " + methodMetadata.getReturnTypeName());
            System.out.println("Method IsAbstract: " + methodMetadata.isAbstract());
            System.out.println("Method IsStatic: " + methodMetadata.isStatic());
            System.out.println("Method IsFinal: " + methodMetadata.isFinal());
            System.out.println("Method IsOverridable: " + methodMetadata.isOverridable());
            System.out.println();
        }
```



# TypeFilter

通过覆盖 `match` 方法定义自己的过滤逻辑。这使得可以根据特定的条件，如类的注解、实现的接口或继承关系等，来决定类是否应该被包含在组件扫描的结果中。

使用方式: 在`@ComponentScan` 中 通过`includeFilters` 和 `excludeFilters` 设置自定义的fliter 来加入到容器中

```java
Filter[] includeFilters() default {};

@interface Filter {
		/**
		 * The type of filter to use.
		 * CUSTOM:标志自定义FilterType 类
		 */
		FilterType type() default FilterType.ANNOTATION;

		/**
		 * The class or classes to use as the filter.
		 * 自定义FilterType 类
     */
		@AliasFor("value")
		Class<?>[] classes() default {};
```



match() 有两个参数

+ MetadataReader metadataReader ： 目标类的元数据
+ MetadataReaderFactory metadataReaderFactory： 用于获取其他类的元数据

```java
 // 创建路径匹配的资源模式解析器
        PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();

        // 创建一个简单的元数据读取器工厂
        SimpleMetadataReaderFactory metadataReaderFactory = new SimpleMetadataReaderFactory();

        // 创建一个注解类型过滤器，用于匹配带有 MyAnnotation 注解的类
        TypeFilter annotationTypeFilter = new AnnotationTypeFilter(MyAnnotation.class);

        // 使用资源模式解析器获取所有匹配指定路径的类文件
        Resource[] resources = resolver.getResources("classpath*:com/xcs/spring/**/*.class");

        // 遍历扫描到的类文件
        for (Resource resource : resources) {
            // 获取元数据读取器
            MetadataReader metadataReader = metadataReaderFactory.getMetadataReader(resource);

            // 使用注解类型过滤器匹配当前类
            boolean match = annotationTypeFilter.match(metadataReader, metadataReaderFactory);

            // 输出扫描到的文件名和匹配结果
            System.out.printf("扫描到的文件: %-20s || 筛选器是否匹配: %s%n", resource.getFile().getName(), match);
        }
```

具体子类：

+ `AnnotationTypeFilter`:  匹配带有指定注解的类。

+ `AssignableTypeFilter` ： 匹配指定类型的子类或实现类

+ `AspectJTypeFilter`： 使用AspectJ表达式进行匹配

+ `RegexPatternTypeFilter`： 使用正则表达式来匹配类的名称。

  
