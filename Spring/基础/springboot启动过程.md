**在springboot项目中每个类的一些属性都在构造类的时候设置好默认值**

# log

分析Springboot 使用的日志框架

```java
	private static final Log logger = LogFactory.getLog(SpringApplication.class);
```

```java
public static Log getLog(String name) {
  // 根据传入的名称来获取对应的日志框架实现
		return LogAdapter.createLog(name);
	}
```

## LogAdapter

在静态代码块中 根据类路径去判断使用哪个日志框架，默认使用的是java.util.logging

```java
static {
		if (isPresent(LOG4J_SPI)) {
			if (isPresent(LOG4J_SLF4J_PROVIDER) && isPresent(SLF4J_SPI)) {
				// log4j-to-slf4j bridge -> we'll rather go with the SLF4J SPI;
				// however, we still prefer Log4j over the plain SLF4J API since
				// the latter does not have location awareness support.
				logApi = LogApi.SLF4J_LAL;
			}
			else {
				// Use Log4j 2.x directly, including location awareness support
				logApi = LogApi.LOG4J;
			}
		}
		else if (isPresent(SLF4J_SPI)) {
			// Full SLF4J SPI including location awareness support
			logApi = LogApi.SLF4J_LAL;
		}
		else if (isPresent(SLF4J_API)) {
			// Minimal SLF4J API without location awareness support
			logApi = LogApi.SLF4J;
		}
		else {
			// java.util.logging as default
			logApi = LogApi.JUL;
		}
	}




public static Log createLog(String name) {
  // 根据静态方法确定好的logApi 来返回具体的日志记录器
		switch (logApi) {
			case LOG4J:
				return Log4jAdapter.createLog(name);
			case SLF4J_LAL:
				return Slf4jAdapter.createLocationAwareLog(name);
			case SLF4J:
				return Slf4jAdapter.createLog(name);
			default:
				// Defensively use lazy-initializing adapter class here as well since the
				// java.logging module is not present by default on JDK 9. We are requiring
				// its presence if neither Log4j nor SLF4J is available; however, in the
				// case of Log4j or SLF4J, we are trying to prevent early initialization
				// of the JavaUtilLog adapter - e.g. by a JVM in debug mode - when eagerly
				// trying to parse the bytecode for all the cases of this switch clause.
				return JavaUtilAdapter.createLog(name);
		}
	}
```

# Run

```java
/**
 * @Author peilizhi
 * @Date 2021/8/29 16:48
 **/
@SpringBootApplication(exclude = {DruidDataSourceAutoConfigure.class})
@MapperScan(basePackages = "com.huochai.domain.testDomain.mapper")
public class TestApplication {
    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
      // 高级用法，先创建SpringApplication ，之后通过setter 设置一些属性，之后执行run方法
    }
}
```

TestApplication  的classLoader是AppClassLoader

# 创建SpringApplication

SpringApplication 可用于从 Java 主方法引导和启动 Spring 应用程序的类。默认情况下，类将执行以下步骤来引导您的应用程序：

+ 创建一个适当的ApplicationContext实例（取决于您的类路径）
+ 注册一个CommandLinePropertySource以将命令行参数公开为 Spring 属性
+ 刷新应用程序上下文，加载所有单例 bean
+ 触发任何CommandLineRunner bean

SpringApplication可以从各种不同的来源读取 bean。通常建议使用单个@Configuration类来引导您的应用程序，但是，您也可以从以下位置设置sources ：

+ AnnotatedBeanDefinitionReader要加载的全限定类名

+ XmlBeanDefinitionReader加载的 XML 资源的位置，或者GroovyBeanDefinitionReader加载的 groovy 脚本的位置

+ ClassPathBeanDefinitionScanner要扫描的包的名称

  ```java
   // 单独查看BeanConfig 能够注入的bean
   AnnotationConfigApplicationContext applicationContext2 = new AnnotationConfigApplicationContext(BeanConfig.class);
  // 实际上是使用AnnotatedBeanDefinitionReader 来加载配置类
  ```

  

```java
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
		return new SpringApplication(primarySources).run(args);
}
```

## 初始化属性

因为SpringApplication中有多个属性，需要先设置好值，之后使用的时候能够直接获取属性就可以

```java
// 初始化一些属性
/**
* resourceLoader:null
* primarySources:主要的 bean 来源 ,实际就是run方法里面指定的类
**/
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
  	// 设置 Web 应用程序类型：通过判断类加载器中是否有特定的类的确定web类型
		this.webApplicationType = WebApplicationType.deduceFromClasspath();
		setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
  	// 设置主启动类似，找含 “main”的否则就是null 
		this.mainApplicationClass = deduceMainApplicationClass();
	}
```

### this.webApplicationType

webApplicationType类型有：

+ NONE：该应用程序不应作为 Web 应用程序运行，也不应启动嵌入式 Web 服务器。
+ SERVLET：该应用程序应作为基于 servlet 的 Web 应用程序运行，并应启动嵌入式 servlet Web 服务器。（默认是）
+ REACTIVE：该应用程序应作为响应式 Web 应用程序运行，并应启动嵌入式响应式 Web 服务器

**是通过判断类加载器中是否有特定的类的确定webApplicationType类型**

```java
static WebApplicationType deduceFromClasspath() {
  // 判断是否能加载哪些类，来确定应用程序的类型
		if (ClassUtils.isPresent(WEBFLUX_INDICATOR_CLASS, null) && !ClassUtils.isPresent(WEBMVC_INDICATOR_CLASS, null)
				&& !ClassUtils.isPresent(JERSEY_INDICATOR_CLASS, null)) {
			return WebApplicationType.REACTIVE;
		}
		for (String className : SERVLET_INDICATOR_CLASSES) {
			if (!ClassUtils.isPresent(className, null)) {
				return WebApplicationType.NONE;
			}
		}
		return WebApplicationType.SERVLET;
	}
```

—— 有org.springframework.web.reactive.DispatcherHandler但是没有org.springframework.web.servlet.DispatcherServlet 没有org.glassfish.jersey.servlet.ServletContainer的话，需要加载嵌入式响应式 Web 服务器

—— 如果应用程序不包含Servlet和ConfigurableWebApplicationContext则为普通应用程序。

——其他情况则为基于servlet的web应用，需加载并启动内嵌的web web服务。

### setInitializers

#### ApplicationContextInitializer

ApplicationContextInitializer 用于在刷新容器之前供我们对ConfigurableApplicationContext实例进行设置

```java
public interface ApplicationContextInitializer<C extends ConfigurableApplicationContext> {

	/**
	 * Initialize the given application context.
	 * @param applicationContext the application to configure
	 */
	void initialize(C applicationContext);
}

```

在一个Springboot应用中，classpath上会包含很多jar包，有些jar包需要在ConfigurableApplicationContext#refresh()调用之前对应用上下文做一些初始化动作，因此它们会提供自己的ApplicationContextInitializer实现类，然后放在自己的META-INF/spring.factories属性文件中，这样相应的ApplicationContextInitializer实现类就会被SpringApplication#initialize发现

+ **DelegatingApplicationContextInitializer**: 使用环境属性context.initializer.classes指定的初始化器(initializers)进行初始化工作，如果没有指定则什么都不做。
+ **ContextIdApplicationContextInitializer**:设置Spring应用上下文的ID,会参照环境属性。
+ **ConfigurationWarningsApplicationContextInitializer**: 对于一般配置错误在日志中作出警告
+ **ServerPortInfoApplicationContextInitializer** : 将内置servlet容器实际使用的监听端口写入到Environment环境属性中。这样属性local.server.port就可以直接通过@Value注入到测试中，或者通过环境属性Environment获取。

#### 自定义ApplicationContextInitializer

1. 先自定义ApplicationContextInitializer 接口

   ```java
   /**
    * @author peilizhi
    * @date 2022/9/19 20:55
    **/
   // 数字越小越优先执行
   @Order(10)
   public class MyApplicationContextInitializer implements ApplicationContextInitializer {
       /**
        * Initialize the given application context.
        *
        * @param applicationContext the application to configure
        */
       @Override
       public void initialize(ConfigurableApplicationContext applicationContext) {
           System.out.println("MyApplicationContextInitializer = " + applicationContext);
       }
   }
   
   ```

   

2. 加入到SpringApplication 里面

   1. 调用springApplication#addInitializers
   2. 在application.properties 添加context.initializer.classes = com.huochai.ApplicationContextInitializer
   3. 在项目下的resources下新建META-INF文件夹，文件夹下新建spring.factories文件(类似容器引入其他jar包中的ApplicationContextInitializer)

#### 添加ApplicationContextInitializer集合

```java
setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
```

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
  // 获取类加载器：AppClassLoader
		ClassLoader classLoader = getClassLoader();
		// Use names and ensure unique to protect against duplicates
  	// 获取所有MATE-INF 中spring.factories中定义的 ApplicationContextInitializer 类名
		Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
  	// 实例化ApplicationContextInitializer 对象 
		List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
   // 排序 
		AnnotationAwareOrderComparator.sort(instances);
		return instances;
	}
```

**SpringFactoriesLoader#loadFactoryNames**

这个接口很重要，后面有很多使用这个接口来获取 所有MATE-INF 中spring.factories中的定义

```java
public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
  // 获取类的全类名
		String factoryTypeName = factoryType.getName();
  // 从spring.factories 获取类名，没有就返回空集合
		return loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
	}

private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
    // 从缓存中获取，拿到就直接返回
		MultiValueMap<String, String> result = cache.get(classLoader);
		if (result != null) {
			return result;
		}

		try {
			Enumeration<URL> urls = (classLoader != null ?
					classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
			result = new LinkedMultiValueMap<>();
			while (urls.hasMoreElements()) {
        // 获取所有jar中 MATE-INF下spring.factories的url
				URL url = urls.nextElement();
				UrlResource resource = new UrlResource(url);
        // 解析参数
				Properties properties = PropertiesLoaderUtils.loadProperties(resource);
				for (Map.Entry<?, ?> entry : properties.entrySet()) {
					String factoryTypeName = ((String) entry.getKey()).trim();
					for (String factoryImplementationName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
						result.add(factoryTypeName, factoryImplementationName.trim());
					}
				}
			}
      // 放入缓存中
			cache.put(classLoader, result);
			return result;
		}
		catch (IOException ex) {
			throw new IllegalArgumentException("Unable to load factories from location [" +
					FACTORIES_RESOURCE_LOCATION + "]", ex);
		}
	}

	@SuppressWarnings("unchecked")
	private static <T> T instantiateFactory(String factoryImplementationName, Class<T> factoryType, ClassLoader classLoader) {
		try {
			Class<?> factoryImplementationClass = ClassUtils.forName(factoryImplementationName, classLoader);
			if (!factoryType.isAssignableFrom(factoryImplementationClass)) {
				throw new IllegalArgumentException(
						"Class [" + factoryImplementationName + "] is not assignable to factory type [" + factoryType.getName() + "]");
			}
			return (T) ReflectionUtils.accessibleConstructor(factoryImplementationClass).newInstance();
		}
		catch (Throwable ex) {
			throw new IllegalArgumentException(
				"Unable to instantiate factory class [" + factoryImplementationName + "] for factory type [" + factoryType.getName() + "]",
				ex);
		}
	}
```

<img src="/Users/peilizhi/Library/Application Support/typora-user-images/image-20220522112259022.png" alt="image-20220522112259022" style="zoom:50%;" />

### setListeners

同设置ApplicationContextInitializer集合

将获取所有ApplicationListener（其他jar下META-INF/spring.factories 里面配置的ApplicationListener）

用于在Springboot 启动的各个环节执行不同的动作，**参考观察者模式**

```java
@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {

	/**
	 * Handle an application event.
	 * @param event the event to respond to
	 */
	void onApplicationEvent(E event);

}
```

# SpringApplication#run

```java
public ConfigurableApplicationContext run(String... args) {
  // 创建一个StopWatch实例，用来记录SpringBoot的启动时间
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
  // 设置一个系统参数，key:java.awt.headless value:true
		configureHeadlessProperty();
		SpringApplicationRunListeners listeners = getRunListeners(args);
		listeners.starting();
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
			ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
			configureIgnoreBeanInfo(environment);
			Banner printedBanner = printBanner(environment);
			context = createApplicationContext();
			exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
			prepareContext(context, environment, listeners, applicationArguments, printedBanner);
			refreshContext(context);
			afterRefresh(context, applicationArguments);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
			}
			listeners.started(context);
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, listeners);
			throw new IllegalStateException(ex);
		}

		try {
			listeners.running(context);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}
```

## configureHeadlessProperty

设置系统属性

SYSTEM_PROPERTY_JAVA_AWT_HEADLESS = “java.awt.headless”，用于记录java.awt.headless的；java.awt.headless是J2SE的一种模式用于在缺少显示屏、键盘或者鼠标时的系统配置，**true:表示缺少这些的时候也能启动**

```java
System.setProperty(SYSTEM_PROPERTY_JAVA_AWT_HEADLESS,
				System.getProperty(SYSTEM_PROPERTY_JAVA_AWT_HEADLESS, Boolean.toString(this.headless)));
```

### System.getProperty（param1,param2）

获取param1 作为key的值，获取不到的话就拿param2.都是设置的是系统属性

### System.setProperty（key,value）

记录key-value 到系统属性中

## getRunListeners

返回监听的集合

```java
private SpringApplicationRunListeners getRunListeners(String[] args) {
		Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
		return new SpringApplicationRunListeners(logger,
				getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args));
	}

// 最终listeners集合里面只有EventPublishingRunListener这一个实例
// SpringApplicationRunListeners 本质是多个SpringApplicationRunListener 的集合
SpringApplicationRunListeners(Log log, Collection<? extends SpringApplicationRunListener> listeners) {
		this.log = log;
		this.listeners = new ArrayList<>(listeners);
	}
```

### getSpringFactoriesInstances

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
    // 获取类加载器
		ClassLoader classLoader = getClassLoader();
		// Use names and ensure unique to protect against duplicates
   	// type = SpringApplicationRunListener.class
  	//  从类路径中获取SpringApplicationRunListener，并且根据优先级排序
		Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
		List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
		AnnotationAwareOrderComparator.sort(instances);
		return instances;
	}
```

**SpringApplicationRunListener**

SpringApplication run方法的侦听器,这个接口定义了在springboot启动的

,在run方法执行的不同时期有不同的监听事件。

```java
public interface SpringApplicationRunListener {

	/**
	 *在 run 方法第一次启动时立即调用。可用于非常早的初始化
	 */
	default void starting() {
	}

	/**
	 * 一旦准备好环境，但在创建ApplicationContext之前调用
	 */
	default void environmentPrepared(ConfigurableEnvironment environment) {
	}

	/**
	 * 在创建和准备ApplicationContext后，但在加载源之前调用。
	 */
	default void contextPrepared(ConfigurableApplicationContext context) {
	}

	/**
	 * 在加载应用程序上下文但在刷新之前调用
	 */
	default void contextLoaded(ConfigurableApplicationContext context) {
	}

	/**
	 * 上下文已刷新，应用程序已启动，但尚未调用CommandLineRunners和ApplicationRunners 
	 */
	default void started(ConfigurableApplicationContext context) {
	}

	/**
	 * 在 run 方法完成之前立即调用，此时应用程序上下文已刷新并且所有CommandLineRunners和ApplicationRunners已被调用。
	 */
	default void running(ConfigurableApplicationContext context) {
	}

	/**
	 * 当运行应用程序发生故障时调用。
	 */
	default void failed(ConfigurableApplicationContext context, Throwable exception) {
	}
}
```

## Start事件

### 创建SpringApplicationRunListeners

```java
// 
public class EventPublishingRunListener implements SpringApplicationRunListener, Ordered {
  private final SpringApplication application;

	private final String[] args;

	private final SimpleApplicationEventMulticaster initialMulticaster;

	public EventPublishingRunListener(SpringApplication application, String[] args) {
		this.application = application;
		this.args = args;
    
		this.initialMulticaster = new SimpleApplicationEventMulticaster();
    // 将所有ApplicationListener 注册到广播中心，后续发布事件的时候就是这个ApplicationListener 来处理
		for (ApplicationListener<?> listener : application.getListeners()) {
			this.initialMulticaster.addApplicationListener(listener);
		}
	}
}
```

### 发布start事件

采用观察者模式，当有事件变更的时候，就调用观察者对应的处理事件的接口

```java
void starting() {
  // 遍历所有监听器，执行start事件
  // listeners中只有EventPublishingRunListener
		for (SpringApplicationRunListener listener : this.listeners) {
			listener.starting();
		}
	}
```

EventPublishingRunListener#start

```java
@Override
	public void starting() {
    // 广播，发布ApplicationStartingEvent事件
		this.initialMulticaster.multicastEvent(new ApplicationStartingEvent(this.application, this.args));
	}

@Override
	public void multicastEvent(ApplicationEvent event) {
		multicastEvent(event, resolveDefaultEventType(event));
	}


@Override
	public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
		ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
		Executor executor = getTaskExecutor();
    // 获取所有监听当前事件的listener
		for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
			if (executor != null) {
				executor.execute(() -> invokeListener(listener, event));
			}
			else {
        // 没有配置执行器的话，就直接执行
				invokeListener(listener, event);
			}
		}
	}


// 获取合适的监听器
protected Collection<ApplicationListener<?>> getApplicationListeners(
			ApplicationEvent event, ResolvableType eventType) {

		Object source = event.getSource();
		Class<?> sourceType = (source != null ? source.getClass() : null);
		ListenerCacheKey cacheKey = new ListenerCacheKey(eventType, sourceType);

		// Quick check for existing entry on ConcurrentHashMap...
		ListenerRetriever retriever = this.retrieverCache.get(cacheKey);
		if (retriever != null) {
			return retriever.getApplicationListeners();
		}

		if (this.beanClassLoader == null ||
				(ClassUtils.isCacheSafe(event.getClass(), this.beanClassLoader) &&
						(sourceType == null || ClassUtils.isCacheSafe(sourceType, this.beanClassLoader)))) {
			// Fully synchronized building and caching of a ListenerRetriever
			synchronized (this.retrievalMutex) {
				retriever = this.retrieverCache.get(cacheKey);
				if (retriever != null) {
					return retriever.getApplicationListeners();
				}
				retriever = new ListenerRetriever(true);
        // 根据事件类型，从所有默认监听器中过滤合适的
				Collection<ApplicationListener<?>> listeners =
						retrieveApplicationListeners(eventType, sourceType, retriever);
				this.retrieverCache.put(cacheKey, retriever);
				return listeners;
			}
		}
		else {
			// No ListenerRetriever caching -> no synchronization necessary
			return retrieveApplicationListeners(eventType, sourceType, null);
		}
	}


protected void invokeListener(ApplicationListener<?> listener, ApplicationEvent event) {
  // 获取事件异常处理器
		ErrorHandler errorHandler = getErrorHandler();
		if (errorHandler != null) {
			try {
				doInvokeListener(listener, event);
			}
			catch (Throwable err) {
				errorHandler.handleError(err);
			}
		}
		else {
			doInvokeListener(listener, event);
		}
	}



private void doInvokeListener(ApplicationListener listener, ApplicationEvent event) {
		try {
      // 手动调用监听器的处理逻辑【同步事件】
			listener.onApplicationEvent(event);
		}
		catch (ClassCastException ex) {
			String msg = ex.getMessage();
			if (msg == null || matchesClassCastMessage(msg, event.getClass())) {
				// Possibly a lambda-defined listener which we could not resolve the generic event type for
				// -> let's suppress the exception and just log a debug message.
				Log logger = LogFactory.getLog(getClass());
				if (logger.isTraceEnabled()) {
					logger.trace("Non-matching event type for listener: " + listener, ex);
				}
			}
			else {
				throw ex;
			}
		}
	}
```

## SpringApplication参数

```java
ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);

public DefaultApplicationArguments(String... args) {
		Assert.notNull(args, "Args must not be null");
		this.source = new Source(args);
		this.args = args;
	}
```

## 创建环境

```java
private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners,
			ApplicationArguments applicationArguments) {
		// Create and configure the environment
  // StandardServletEnvironment 环境
		ConfigurableEnvironment environment = getOrCreateEnvironment();
  // 设置profile ：激活的哪个环境
		configureEnvironment(environment, applicationArguments.getSourceArgs());
		ConfigurationPropertySources.attach(environment);
		listeners.environmentPrepared(environment);
		bindToSpringApplication(environment);
		if (!this.isCustomEnvironment) {
			environment = new EnvironmentConverter(getClassLoader()).convertEnvironmentIfNecessary(environment,
					deduceEnvironmentClass());
		}
		ConfigurationPropertySources.attach(environment);
		return environment;
	}

```

### 根据web程序类型创建环境

```java
private ConfigurableEnvironment getOrCreateEnvironment() {
		if (this.environment != null) {
			return this.environment;
		}
		switch (this.webApplicationType) {
		case SERVLET:
			return new StandardServletEnvironment();
		case REACTIVE:
			return new StandardReactiveWebEnvironment();
		default:
			return new StandardEnvironment();
		}
	}
```

## environmentPrepared事件

```java
void environmentPrepared(ConfigurableEnvironment environment) {
		for (SpringApplicationRunListener listener : this.listeners) {
			listener.environmentPrepared(environment);
		}
	}
```

#### 读取配置文件

其中ConfigFileApplicationListener是解析配置文件的，解析完成之后加入到**Environment 中的MutablePropertySources中**

ConfigFileApplicationListener#onApplicationEvent

```java
@Override
	public void onApplicationEvent(ApplicationEvent event) {
		if (event instanceof ApplicationEnvironmentPreparedEvent) {
      // 处理environmentPrepared 事件
			onApplicationEnvironmentPreparedEvent((ApplicationEnvironmentPreparedEvent) event);
		}
		if (event instanceof ApplicationPreparedEvent) {
			onApplicationPreparedEvent(event);
		}
	}


private void onApplicationEnvironmentPreparedEvent(ApplicationEnvironmentPreparedEvent event) {
  // 获取MATE-INF 下spring.factories 中的 EnvironmentPostProcessor类
		List<EnvironmentPostProcessor> postProcessors = loadPostProcessors();
  // 加入本类
		postProcessors.add(this);
		AnnotationAwareOrderComparator.sort(postProcessors);
		for (EnvironmentPostProcessor postProcessor : postProcessors) {
			postProcessor.postProcessEnvironment(event.getEnvironment(), event.getSpringApplication());
		}
	}

// 执行ConfigFileApplicationListener 的postProcessEnvironment方法
@Override
	public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) {
		addPropertySources(environment, application.getResourceLoader());
	}

protected void addPropertySources(ConfigurableEnvironment environment, ResourceLoader resourceLoader) {
		RandomValuePropertySource.addToEnvironment(environment);
  // 创建解析器，并执行加载方法
		new Loader(environment, resourceLoader).load();
	}
```

##### load

```java
void load() {
			FilteredPropertySource.apply(this.environment, DEFAULT_PROPERTIES, LOAD_FILTERED_PROPERTY,
					(defaultProperties) -> {
						this.profiles = new LinkedList<>();
						this.processedProfiles = new LinkedList<>();
						this.activatedProfiles = false;
						this.loaded = new LinkedHashMap<>();
            // 创建默认default的 profiles
						initializeProfiles();
						while (!this.profiles.isEmpty()) {
							Profile profile = this.profiles.poll();
							if (isDefaultProfile(profile)) {
								addProfileToEnvironment(profile.getName());
							}
              
							load(profile, this::getPositiveProfileFilter,
									addToLoaded(MutablePropertySources::addLast, false));
							this.processedProfiles.add(profile);
						}
            
            // 消费者是addToLoaded返回的对象
						load(null, this::getNegativeProfileFilter, addToLoaded(MutablePropertySources::addFirst, true));
						addLoadedPropertySources();
						applyActiveProfiles(defaultProperties);
					});
		}

private void load(Profile profile, DocumentFilterFactory filterFactory, DocumentConsumer consumer) {
			getSearchLocations().forEach((location) -> {
				boolean isFolder = location.endsWith("/");
				Set<String> names = isFolder ? getSearchNames() : NO_SEARCH_NAMES;
				names.forEach((name) -> load(location, name, profile, filterFactory, consumer));
			});
		}

// 获取可解析配置文件的目录
private Set<String> getSearchLocations() {
			if (this.environment.containsProperty(CONFIG_LOCATION_PROPERTY)) {
				return getSearchLocations(CONFIG_LOCATION_PROPERTY);
			}
			Set<String> locations = getSearchLocations(CONFIG_ADDITIONAL_LOCATION_PROPERTY);
			locations.addAll(
					asResolvedSet(ConfigFileApplicationListener.this.searchLocations, DEFAULT_SEARCH_LOCATIONS));
			return locations;
		}
```

有四种路径可以定义配置文件路径

1. file:./config/
2. file:./
3. classpath:/config/
4. classpath:/

这如果这几个目录下的有相同的属性，那么会使用最后加载的属性

```java
// 获取文件名称，如果不指定spring.config.name 参数的话，就会默认使用application作为文件名
private Set<String> getSearchNames() {
			if (this.environment.containsProperty(CONFIG_NAME_PROPERTY)) {
				String property = this.environment.getProperty(CONFIG_NAME_PROPERTY);
				return asResolvedSet(property, null);
			}
			return asResolvedSet(ConfigFileApplicationListener.this.names, DEFAULT_NAMES);
		}
```

```java
private void load(String location, String name, Profile profile, DocumentFilterFactory filterFactory,
				DocumentConsumer consumer) {
			if (!StringUtils.hasText(name)) {
				for (PropertySourceLoader loader : this.propertySourceLoaders) {
					if (canLoadFileExtension(loader, location)) {
						load(loader, location, profile, filterFactory.getDocumentFilter(profile), consumer);
						return;
					}
				}
				throw new IllegalStateException("File extension of config file location '" + location
						+ "' is not known to any PropertySourceLoader. If the location is meant to reference "
						+ "a directory, it must end in '/'");
			}
			Set<String> processed = new HashSet<>();
  // 有两种资源解析器：PropertiesPropertySourceLoader、YamlPropertySourceLoader
			for (PropertySourceLoader loader : this.propertySourceLoaders) {
        // 遍历文件名后缀
				for (String fileExtension : loader.getFileExtensions()) {
					if (processed.add(fileExtension)) {
            // 这里location + name, "." + fileExtension 组装文件实际地址
						loadForFileExtension(loader, location + name, "." + fileExtension, profile, filterFactory,
								consumer);
					}
				}
			}
		}
```

YamlPropertySourceLoader负责解析 yam,yaml后缀的文件

PropertiesPropertySourceLoader负责解析properties，xml后缀的文件

```java
private void loadForFileExtension(PropertySourceLoader loader, String prefix, String fileExtension,
				Profile profile, DocumentFilterFactory filterFactory, DocumentConsumer consumer) {
			DocumentFilter defaultFilter = filterFactory.getDocumentFilter(null);
			DocumentFilter profileFilter = filterFactory.getDocumentFilter(profile);
  		// 文件中指定了profile的时候
			if (profile != null) {
				// Try profile-specific file & profile section in profile file (gh-340)
				String profileSpecificFile = prefix + "-" + profile + fileExtension;
				load(loader, profileSpecificFile, profile, defaultFilter, consumer);
				load(loader, profileSpecificFile, profile, profileFilter, consumer);
				// Try profile specific sections in files we've already processed
				for (Profile processedProfile : this.processedProfiles) {
					if (processedProfile != null) {
						String previouslyLoaded = prefix + "-" + processedProfile + fileExtension;
						load(loader, previouslyLoaded, profile, profileFilter, consumer);
					}
				}
			}
			// Also try the profile-specific section (if any) of the normal file
			load(loader, prefix + fileExtension, profile, profileFilter, consumer);
		}
```

```java
private void load(PropertySourceLoader loader, String location, Profile profile, DocumentFilter filter,
				DocumentConsumer consumer) {
			try {
				Resource resource = this.resourceLoader.getResource(location);
				if (resource == null || !resource.exists()) {
					if (this.logger.isTraceEnabled()) {
						StringBuilder description = getDescription("Skipped missing config ", location, resource,
								profile);
						this.logger.trace(description);
					}
					return;
				}
				if (!StringUtils.hasText(StringUtils.getFilenameExtension(resource.getFilename()))) {
					if (this.logger.isTraceEnabled()) {
						StringBuilder description = getDescription("Skipped empty config extension ", location,
								resource, profile);
						this.logger.trace(description);
					}
					return;
				}
				String name = "applicationConfig: [" + location + "]";
        // 获取文件，里面有这种文件配置
				List<Document> documents = loadDocuments(loader, name, resource);
				if (CollectionUtils.isEmpty(documents)) {
					if (this.logger.isTraceEnabled()) {
						StringBuilder description = getDescription("Skipped unloaded config ", location, resource,
								profile);
						this.logger.trace(description);
					}
					return;
				}
        
				List<Document> loaded = new ArrayList<>();
				for (Document document : documents) {
					if (filter.match(document)) {
						addActiveProfiles(document.getActiveProfiles());
						addIncludedProfiles(document.getIncludeProfiles());
						loaded.add(document);
					}
				}
				Collections.reverse(loaded);
				if (!loaded.isEmpty()) {
					loaded.forEach((document) -> consumer.accept(profile, document));
					if (this.logger.isDebugEnabled()) {
						StringBuilder description = getDescription("Loaded config file ", location, resource, profile);
						this.logger.debug(description);
					}
				}
			}
			catch (Exception ex) {
				throw new IllegalStateException("Failed to load property source from location '" + location + "'", ex);
			}
		}




private DocumentConsumer addToLoaded(BiConsumer<MutablePropertySources, PropertySource<?>> addMethod,
				boolean checkForExisting) {
  
  // 消费者的方法
			return (profile, document) -> {
				if (checkForExisting) {
					for (MutablePropertySources merged : this.loaded.values()) {
						if (merged.contains(document.getPropertySource().getName())) {
							return;
						}
					}
				}
       
				MutablePropertySources merged = this.loaded.computeIfAbsent(profile,
						(k) -> new MutablePropertySources());
				addMethod.accept(merged, document.getPropertySource());
			};
		}
// addMethod.accept方法
public void addFirst(PropertySource<?> propertySource) {
		removeIfPresent(propertySource);
   // MutablePropertySources的propertySourceList
		this.propertySourceList.add(0, propertySource);
	}
```







