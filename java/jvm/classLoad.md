

![image-20241016005744975](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410160057074.png)

# 作用

ClassLoader 它是用来加载 Class 的。它负责将 Class 的字节码形式转换成内存形式的 Class 对象。字节码可以来自于磁盘文件 *.class，也可以是 jar 包里的 *.class，也可以来自远程服务器提供的字节流，字节码的本质就是一个字节数组 []byte

很多字节码加密技术就是依靠定制 ClassLoader 来实现的。先使用工具对字节码文件进行加密，运行时使用定制的 ClassLoader 先解密文件内容再加载这些解密后的字节码。 

每个类上都含有 classLoader ,用于指定当前类由哪个ClassLoader 来解析

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202402220055277.png" alt="image-20240222005505177" style="zoom:50%;" />

*ClassLoader* ： 是抽象类，定义了加载class的流程

SecureClassLoader：是抽象类，类加载过程中增加了安全管理的功能。

URLClassLoader：提供了从非系统类路径或者网络上的特定URL位置加载类位置加载类的功能，无需在启动时就加载所有依赖。通过使用不同的URLClassLoader实例可以实现不同模块间的类加载隔离。每个类加载器只能访问其加载过的类，从而支持类加载沙箱机制。

```java
 // 指定要加载类的URL位置
        URL url = new URL("file:/path/to/your/classes");
        
        // 创建URLClassLoader实例
        URLClassLoader classLoader = new URLClassLoader(new URL[]{url}, Thread.currentThread().getContextClassLoader());

        // 使用URLClassLoader加载类
        Class<?> clazz = classLoader.loadClass("com.example.YourClass");
        // 可以通过反射创建类的实例
        Object instance = clazz.getDeclaredConstructor().newInstance();
        // 使用完后关闭资源（注意：不是所有版本的URLClassLoader都有close方法）
        if (classLoader instanceof Closeable) {
            ((Closeable) classLoader).close();
        }
```



最关键的三个作用：

+ 双亲委派
+ 类缓存
+ 沙箱保护机制

# 双亲委派

JVM 中内置了三个重要的 ClassLoader，分别是 **BootstrapClassLoader**、**ExtensionClassLoader** 和 **AppClassLoader**。

BootstrapClassLoader 负责加载 JVM 运行时核心类，这些类位于 JAVA_HOME/lib/rt.jar 文件中，我们常用内置库 java.xxx.* 都在里面，比如 java.util.*、java.io.*、java.nio.*、java.lang.* 等等。这个 ClassLoader 比较特殊，它是由 C 代码实现的，我们将它称之为「根加载器」。java 想要打印 ExtensionClassLoader 的父加载器的时候会打印出null。

```sh
# 修改BootstrapClassLoader 加载路径，会加载子目录
-D sun.boot.class.path=加载路径
```

ExtensionClassLoader 负责加载 JVM 扩展类，比如 swing 系列、内置的 js 引擎、xml 解析器 等等，这些库名通常以 javax 开头，它们的 jar 包位于 JAVA_HOME/lib/ext/*.jar 中，有很多 jar 包。

```sh
# 修改ExtensionClassLoader 加载路径，会加载子目录
-D java.ext.dirs=加载路径
```

AppClassLoader 才是直接面向我们用户的加载器，它会加载 Classpath 环境变量里定义的路径中的 jar 包和目录。我们自己编写的代码以及使用的第三方 jar 包通常都是由它来加载的。

```sh
# 修改AppClassLoader 加载路径,不会加载子目录。在加载子目录中的资源文件时要指定相对目录
-D java.class.path=加载路径
```

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202402202347323.png" alt="image-20240220234715273" style="zoom: 33%;" />

**所有parent是null的都会默认使用BootstrapClassLoader 来作为父加载类**

当一个类加载器收到类加载请求时，它**不会自己先加载**，而是**先把请求委派给父类加载器**去完成。每一层都如此，最终所有的加载请求都会传递到最顶层的**启动类加载器（Bootstrap ClassLoader）**。只有当父类加载器反馈无法加载时，子加载器才会尝试自己加载。



优势：

1. 避免类重复加载
2. 保证java 核心类的安全
3. 保证类的唯一性  JVM 判断两个类是否相同，除了看**全限定类名**，还要看**加载该类的类加载器**



打破双亲委派机制：

1. tomcat 为每个web 应用创建独立的类加载器。优先加载 /WEB-INF/classes 下的类，打破双亲委派机制



# 类缓存

每个类加载器都对他加载过的类有个缓存（流程的第一步）

# 沙箱保护机制

不允许加载jdk内部类，不允许加载java. 开头的类

findClass() -> defineClass() -> preDefineClass()

```java
private ProtectionDomain preDefineClass(String name,
                                            ProtectionDomain pd)
    {
        if (!checkName(name))
            throw new NoClassDefFoundError("IllegalName: " + name);

        // Note:  Checking logic in java.lang.invoke.MemberName.checkForTypeAlias
        // relies on the fact that spoofing is impossible if a class has a name
        // of the form "java.*"
        if ((name != null) && name.startsWith("java.")) {
            throw new SecurityException
                ("Prohibited package name: " +
                 name.substring(0, name.lastIndexOf('.')));
        }
        if (pd == null) {
            pd = defaultDomain;
        }

        if (name != null) checkCerts(name, pd.getCodeSource());

        return pd;
    }
```



# 类加载过程

类加载分为三个步骤：**加载**，**连接**，**初始化**；

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202402210018331.png" alt="image-20240221001837275" style="zoom:50%;" />

## 加载

java.lang.ClassLoader的loadClass()

1. 通过一个类的全限定名来获取定义此类的**二进制字节流**（并没有指明要从一个Class文件中获取，可以从其他渠道，譬如：网络、动态生成、数据库等）
2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构；
3. 在内存中生成一个代表这个类的java.lang.Class对象，作为**方法区**这个类的各种数据的访问入口；

## 验证

确保类能够被虚拟机使用

1. 文件格式验证：验证字节流是否符合Class文件格式的规范；例如：是否以魔术0xCAFEBABE开头、主次版本号是否在当前虚拟机的处理范围之内、常量池中的常量是否有不被支持的类型。
2. 元数据验证：对字节码描述的信息进行语义分析（注意：对比javac编译阶段的语义分析），以保证其描述的信息符合Java语言规范的要求；例如：这个类是否有父类，除了java.lang.Object之外。
3. 字节码验证：通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的。
4. 符号引用验证：确保***解析***动作能正确执行。

但是这个步骤不是必须的，-Xverifynone参数来关闭大部分的类验证措施，以缩短虚拟机类加载的时间。

## 准备

准备阶段是正式为**类变量**分配内存并设置类变量初始值的阶段，这些变量所使用的内存都将在***方法区***中进行分配。这时候进行内存分配的仅包括类变量（被static修饰的变量），而不包括实例变量；实例变量将会在对象实例化时随着对象一起分配在堆中。

概括为：**分内存、定初值**

| byte    | 0     |
| ------- | ----- |
| short   | 0     |
| int     | 0     |
| long    | 0     |
| float   | 0.0   |
| double  | 0.0   |
| char    | ""    |
| boolean | flase |

```java
# 在准备阶段 name 的值是 "" 
private  static  String name = "peilizhi";

# 在准备阶段 name2 的值是 "peilizhi"
private   final static  String name2 = "peilizhi";
```

## 解析

解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。在准备阶段的时候还不知道具体的内存地址，只是一个符号标志，现在变成真正的内存地址

解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符7类符号引用进行。

## 初始化

所有类变量初始化语句和静态代码块都会在编译时被前端编译器放在收集器里头，存放到一个特殊的方法中，这个方法就是<clinit>方法

执行代码设置去初始化类变量和其他资源，或者说：初始化阶段是执行类构造器`<clinit>()`方法的过程.

`<clinit>()`方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块static{}中的语句合并产生的，编译器收集的顺序是由语句在源文件中出现的顺序所决定的，静态语句块只能访问到定义在静态语句块之前的变量，定义在它之后的变量，在前面的静态语句块可以赋值，但是不能访问

```java
public class Test
{
    static
    {
        i=0;
        System.out.println(i);//这句编译器会报错：Cannot reference a field before it is defined（非法向前应用）
    }
    static int i=1;
}
```

解决方法一

```java
public class Test
{
   static int i=1;
    static
    {
        i=0;
        System.out.println(i);
    }
    # 最终 i=0
}
```

()方法与实例构造器`<init>()`方法不同，它不需要显示地调用父类构造器，虚拟机会保证在子类`<init>()`方法执行之前，父类的`<clinit>()`方法方法已经执行完毕,也就意味着父类中定义的静态语句块要优先于子类的变量赋值操作

### 触发初始化（直接初始化）

1. 为一个类型创建一个新的对象实例时（比如new、反射、序列化）
2. 调用一个类型的**静态方法**时（即在字节码中执行invokestatic指令）
3. 调用一个类型或接口的**静态字段**，或者对这些静态字段执行赋值操作时（即在字节码中，执行getstatic或者putstatic指令），不过用final修饰的静态字段除外，它被初始化为一个编译时常量表达式，子类调用父类的静态字段时不是初始化子类
4. 调用JavaAPI中的反射方法时（比如调用java.lang.Class中的方法，或者java.lang.reflect包中其他类的方法）
5. 初始化一个类的子类时（Java虚拟机规范明确要求初始化一个类时，它的超类必须提前完成初始化操作，接口例外）
6. JVM启动包含main方法的启动类时
6. 接口实现类初始化的时候，会触发直接或间接实现的所有接口的初始化。



间接初始化：不会触发初始化

+ 当引用了一个类的静态变量，而该静态变量继承自父类的话，不引起初始化
+ 定义一个类的数组，不会引起该类的初始化；
+ 当引用一个类的的常量时，不会引起该类的初始化



## 卸载

想要回归 jvm 中的class 需要满足三个条件

+ 该类所有的实例都已经被GC，也就是JVM中不存在该Class的任何实例。
+ 加载该类的ClassLoader已经被GC。
+ 该类的java.lang.Class 对象没有在任何地方被引用，如不能在任何地方通过反射访问该类的方法 

满足这些条件之后会就会将方法区的类信息清空

## 代码执行流程

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202402210006926.png" alt="image-20240221000644868" style="zoom: 50%;" />





# 自定义类加载器

只要继承ClassLoader 就可以

根据情景分成两类

一、保留双亲委派机制，需要重写findClass() 方法，定义自己的查询方法

二、不需要双亲委派机制，需要重写loadClass() ,在实现中重新定义查找的方法

```java
public class MyClassLoader extends ClassLoader {
    /**
     * 类的路径
     */
    private String path;
    public MyClassLoader(String path) {
        this.path = path;
    }

    /**
     * 参数是 类的路径
     * @return
     * @throws ClassNotFoundException
     */
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        // 分成两部分
        // 1. 获取类的二进制文件
        // 1.1 自定义的话，对进行进行一些处理，不一定需要读取.class文件
        String filePath = this.path + name + ".class";
        System.out.println("filePath = " + filePath);
        int code;
        try {
            FileInputStream fis = new FileInputStream(filePath);
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            try {
                while ((code = fis.read()) != -1) {
                    bos.write(code);
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            byte[] data = bos.toByteArray();
            bos.close();
            // 2. 通过二进制文件，生成类对象-有沙箱保护机制，不允许加载java.开头包
            return defineClass(name, data, 0, data.length);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

想要重写loadClass() ,不光需要考虑自己特定的类，还需要考虑基础类比如java.lang.Object ,是因为类加载 有懒加载这个特性，需要再使用的时候在进行加载

```java
 @Override
    public Class<?> loadClass(String name) throws ClassNotFoundException {
        System.out.println("loadClass ");
        return findClass(name);
    }
```

修改为 一些基础类还是由父加载器去加载：

```java
@Override
    public Class<?> loadClass(String name) throws ClassNotFoundException {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            // 先自己加载
            Class<?> c = findLoadedClass(name);
            // 失败后交给父加载器
            if (c == null) {
                c = findClass(name);
                long t0 = System.nanoTime();
                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    try {
                        if (this.getParent() != null) {
                            c = this.getParent().loadClass(name);
                        } else {
                            c = findSystemClass(name);
                        }
                    } catch (ClassNotFoundException e) {
                        // ClassNotFoundException thrown if class not found
                        // from the non-null parent class loader
                    }
                }
            }
            return c;
        }
    }
```



```java
MyClassLoader myClassLoader = new MyClassLoader("/Users/peilizhi/IdeaProjects/demo/target/classes/");
Class<?> aClass = myClassLoader.loadClass("com.huochai.demo.load.User");


Class<?> aClass1 = Class.forName("com.huochai.demo.load.User", true, myClassLoader);
```

这两个方法都能获取类，区别就在于第二个方法执行了 ***链接***  。会对静态变量复制，会执行静态代码块。是不太安全的做法。

**loadClass() 方法默认是不执行链接的**

经典错误

```java
public static void main(String[] args) throws ClassNotFoundException, InstantiationException, IllegalAccessException {
        MyClassLoader myClassLoader = new MyClassLoader("/Users/peilizhi/IdeaProjects/demo/target/classes/");
        Class<?> aClass = myClassLoader.loadClass("com.huochai.demo.load.User");
        
        // aClass.getClassLoader() = com.huochai.demo.load.MyClassLoader@63961c42
        System.out.println("aClass.getClassLoader() = " + aClass.getClassLoader());
        User object = (User)aClass.newInstance();
        System.out.println("object = " + object);
    }
```

在User object = (User)aClass.newInstance(); 会报错.

![image-20240221231202339](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202402212312712.png)

虽然aClass 都是 User 类，但是 aClass 的类加载器是 com.huochai.demo.load.MyClassLoader@63961c42，而系统目录下的User 是由AppClassLoader加载器的，**所以他们不是同一个类**



解决方法一：

反射

```java
MyClassLoader myClassLoader = new MyClassLoader("/Users/peilizhi/IdeaProjects/demo/target/classes/");
 Class<?> aClass = myClassLoader.loadClass("com.huochai.demo.load.User");        // 反射执行User 中的sayName()方法
 aClass.getMethod("sayName").invoke(aClass1.newInstance());
```

解决方法二：

SPI

SPI（Service Provider Interface），是JDK内置的一种 服务提供发现机制，可以用来启用框架扩展和替换组件，Java的SPI机制可以为某个接口寻找服务实现。

1. 先编写接口

   ```java
   public interface CalculateService {
       Double calculate(double a, double b);
   }
   ```

2. 编写实现

   ```java
   public class CalculateAServiceImpl implements CalculateService {
       @Override
       public Double calculate(double a, double b) {
           return a + b;
       }
   }
   
   public class CalculateBServiceImpl implements CalculateService {
       @Override
       public Double calculate(double a, double b) {
           return a - b;
       }
   }
   ```

3. 在resources 目录下创建 META-INF/services 目录，并且新建文件，文件名是接口全路径

   <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202402212342571.png" alt="image-20240221234207407" style="zoom:50%;" />

4. 在文件中加入实现类

   ```
   com.huochai.demo.service.CalculateAServiceImpl
   com.huochai.demo.service.CalculateBServiceImpl
   ```

5. 调用

   ```java
    public static void main(String[] args) {
           ServiceLoader<CalculateService> loader = ServiceLoader.load(CalculateService.class);
           loader.forEach(calculateService -> {
               System.out.println(calculateService.calculate(1, 2));
           });
       }
   ```

   执行结果于文件中的接口顺序有关

   ```java
   # load 方法可以指定类加载器，如果特殊制定的话就使用当前线程的类对应的类加载器
   public static <S> ServiceLoader<S> load(Class<S> service,
                                               ClassLoader loader)
       {
           return new ServiceLoader<>(service, loader);
       }
   ```

   

   具体案例

   ```java
   public static void main(String[] args) throws Exception {
           Double salary = 15000.00;
           //使用 URLClassLoader，就不需要在 OADemo 中添加 SPI 的配置文件，直接在 SalaryCaler.jar中添加 SPI 配置文件即可
           //将实现类和SPI 配置文件放在一起，更符合工程化的思想
           while (true) {
               String jarPath1 = "file:/Users/roykingw/lib/SalaryCaler.jar";
               URLClassLoader urlClassLoader1 = new URLClassLoader(new URL[] {new URL(jarPath1)});
               SalaryCalService salaryService1 = getSalaryService(urlClassLoader1);
               System.out.println("应该到手Money:" + salaryService1.cal(salary));
   
               String jarPath2 = "file:/Users/roykingw/lib2/SalaryCaler.jar";
               URLClassLoader urlClassLoader2 = new URLClassLoader(new URL[] {new URL(jarPath2)});
               SalaryCalService salaryService2 = getSalaryService(urlClassLoader2);
               System.out.println("实际到手Money:" + salaryService2.cal(salary));
   
               SalaryCalService salaryCalService3 = getSalaryService(null);
               System.out.println("OA系统中计算的Money:"+salaryCalService3.cal(salary));
               Thread.sleep(5000);
           }
       }
       private static SalaryCalService getSalaryService(ClassLoader classloader){
           ServiceLoader<SalaryCalService> services;
           if(null == classloader){
               services = ServiceLoader.load(SalaryCalService.class);
           }else{
               // ClassLoader c1 = Thread.currentThread().getContextClassLoader();
               // Thread.currentThread().setContextClassLoader(classloader);
               services = ServiceLoader.load(SalaryCalService.class,c1);
               // Thread.currentThread().setContextClassLoader(c1);
   
           }
           SalaryCalService service = null;
           if(null != services){
               //这里只需要拿SPI加载到的第一个实现类
               Iterator<SalaryCalService> iterator = services.iterator();
               if(iterator.hasNext()){
                   service = iterator.next();
               }
           }
           return service;
       }
   ```

   URLClassLoader  可以从支持从jar文件和文件夹获取class。 **类加载器的本质就是从任意位置获取类**

   其中urlClassLoader1，urlClassLoader2 加载的jar 不一致，那么获取到的SalaryCalService就不一致就可以实现不同的功能。

解决方法三：

java  -classpath 

其实idea 启动的时候就使用了 这个指令。 也能将三方包引入到classpath中

![image-20240222004148274](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202402220041333.png)



# 延迟加载

JVM 运行并不是一次性加载所需要的全部类的，它是按需加载，也就是延迟加载。程序在运行的过程中会逐渐遇到很多不认识的新类，这时候就会调用 ClassLoader 来加载这些类。加载完成后就会将 Class 对象存在 ClassLoader 里面，下次就不需要重新加载了。

在调用静态方法的时候，首先这个类肯定是需要被加载的，但是并不会触及这个类的实例字段，那么实例字段的类别 Class 就可以暂时不必去加载

# classLoader 传递性

程序在运行过程中，遇到了一个未知的类，它会选择哪个 ClassLoader 来加载它呢？虚拟机的策略是使用调用者 Class 对象的 ClassLoader 来加载当前未知的类。就是那个方法里面使用未知的类，这个方法属于类A,就使用类A 的加载器来加载未知类

# 钻石依赖

钻石依赖」，是指软件依赖导致同一个软件包的两个版本需要共存而不能冲突。

<img src="https://pic4.zhimg.com/80/v2-7a471cc06fd74bb65984647dbe621a8f_1440w.jpg" alt="img" style="zoom:50%;" />

maven 是这样解决钻石依赖的，它会从多个冲突的版本中选择一个来使用，如果不同的版本之间兼容性很糟糕，那么程序将无法正常编译运行。

使用 ClassLoader 可以解决钻石依赖问题。不同版本的软件包使用不同的 ClassLoader 来加载，**位于不同 ClassLoader 中名称一样的类实际上是不同的类**。就是通过调用者的类加载器来区别不同的版本





类加载问题

```
java: 无法访问org.springframework.boot.SpringApplication
  错误的类文件: /Users/peilizhi/apache-maven-3.6.3/repository/org/springframework/boot/spring-boot/3.0.0/spring-boot-3.0.0.jar!/org/springframework/boot/SpringApplication.class
    类文件具有错误的版本 61.0, 应为 52.0
    请删除该文件或确保该文件位于正确的类路径子目录中。
```

类文件中表明了使用哪个版本的jdk来处理，比如 jdk8 的版本是 52 ，但是 SpringApplication.class 中版本是61，无法识别。

查看class文件版本信息：

1. 先编译
2. 进入class 目录
3. 执行Java -v [class]

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202402202303113.png" alt="image-20240220230258660" style="zoom:50%;" />

