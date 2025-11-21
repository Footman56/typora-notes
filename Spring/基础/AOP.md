#    cglib代理

cglib 是基于父子类来实现的，被代理类是父类，代理类是子类

```java
public static void main(String[] args) {
        UserService userService = new UserService();
        Enhancer enhancer = new Enhancer();
        // 设置目标类
        enhancer.setSuperclass(UserService.class);
        enhancer.setCallback(new MethodInterceptor() {
            @Override
            public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                System.out.println("AOP开始拦截");
                //return methodProxy.invoke(userService, objects);
                return methodProxy.invokeSuper(o, objects);
            }
        });

        UserService proxy = (UserService)enhancer.create();
        proxy.test();
    }
```

# jdk 代理

被代理类必须是接口，使用起来比较麻烦

```java
public class OrderService implements AddressService {
    @Override
    public String sayHello() {
        System.out.println(" hello");
        return "hello";
    }
    public static void main(String[] args) {
        OrderService addressService = new OrderService();
        AddressService object = (AddressService) Proxy.newProxyInstance(AddressService.class.getClassLoader(), new Class[]{AddressService.class}, new InvocationHandler(
        ) {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("proxy = ");
                return method.invoke(addressService, args);
            }
        });
        object.sayHello();
    }
}
```

# ProxyFactory

AOP 封装的代理，不需要关注底层实现，只需完善代理功能。如果实现了接口的话，就使用jdk 代理，否则就是cglib代理

advice 就是增强方法

```java
public static void main(String[] args) {
        UserService userService = new UserService();
        ProxyFactory proxyFactory = new ProxyFactory();
        proxyFactory.setTarget(userService);
        proxyFactory.addAdvice(new MethodInterceptor() {
            @Override
            public Object invoke(MethodInvocation invocation) throws Throwable {
                System.out.println(" 方法前");
                Object result = invocation.proceed();
                System.out.println(" 方法后");
                return result;
            }
        });
        proxyFactory.addAdvice(new MethodBeforeAdvice() {
            @Override
            public void before(Method method, Object[] args, Object target) throws Throwable {
                System.out.println(" before Method" );
            }
        });
        UserService proxy = (UserService) proxyFactory.getProxy();
        proxy.test();
    }
// 方法前
// before Method
// test
// 方法后
```

可以添加多个advice ,多个advice 执行的顺序由添加顺序决定。采用调用链执行方式。

这种不太灵活，是对所有方法都执行增强

```java
 public static void main(String[] args) {
        UserService userService = new UserService();
        ProxyFactory proxyFactory = new ProxyFactory();
        proxyFactory.setTarget(userService);

        Pointcut pointcut = new StaticMethodMatcherPointcut() {
            @Override
            public boolean matches(Method method, Class<?> targetClass) {
                // 对A方法进行增强
                return method.getName().equals("a");
            }
        };
        Advice advice = (MethodInterceptor) invocation -> {
            System.out.println(" 方法前");
            Object result = invocation.proceed();
            System.out.println(" 方法后");
            return result;
        };
        proxyFactory.addAdvisors(new DefaultPointcutAdvisor(pointcut, advice));
        UserService proxy = (UserService) proxyFactory.getProxy();
        proxy.test();
        proxy.a();
    }
```

Advisor 由 pointcut 来决定对那些方法进行增强，advice 决定了增强的方式。

实际上aop的底层实现应该就是创建Advisor 

DefaultAdvisorAutoProxyCreator 是BeanPostProcessor



在springboot 中采用何种方式创建代理对象的：

采用jdk 代理的情况

+ 被代理类是接口  ||  被代理类是 由jdk 代理生成的 ||  含lamdba 表达式的

采用cglib代理

+ 开启优化 ||  非接口



**生成代理对象是由AbstractAutoProxyCreator#postProcessAfterInitialization实现的。 自动配置引入的是AnnotationAwareAspectJAutoProxyCreator，AnnotationAwareAspectJAutoProxyCreator 是AbstractAutoProxyCreator 的子类。**

![image-20241211011033641](https://cdn.jsdelivr.net/gh/Footman56/imageBed2/202412110110717.png)