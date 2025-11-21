# 一、反射

## 1、定义

```
在运行的时候才知道到创建的对象是什么
```

## 2、方法

### a、获取Class

#### I、**使用 Class.forName 静态方法**

```
Class clz = Class.forName("java.lang.String");
需要知道该类的全路径名
```

#### II、**使用 .class 方法**

```java
Class clz = String.class;
需要知道类名
```

#### III、**使用类对象的 getClass()** 

```java
String str = new String("Hello");
Class clz = str.getClass();
```

### b、创建类对象

#### I、通过 Class 对象的 newInstance() 方法

```java
Class clz = Apple.class;
Apple apple = (Apple)clz.newInstance();
只支持无参构造函数
```

#### II、通过 Constructor 对象的 newInstance() 方法

```java
Class clz = Apple.class;
Constructor constructor = clz.getConstructor();
Apple apple = (Apple)constructor.newInstance();
支持多个参数
```



### c、获取方法，属性

```
通过Class类的方法来获取
Field[] fields = clz.getFields();  // 共有属性
Field[] fields = clz.getDeclaredFields(); // 获取所有属性
```



# 二、模型

# 1、分类

```java
常用的对象有	Model，DO，DTO，PO
Model:作为展示的数据模型，通常作为Controller的入参「Controller内完成Model到DO的转换」
DO：对应现实中的业务模型，常用于Service的入参 [ Service内完成DO到PO的转换]
DTO:常用于数据传输，类型RPC调用的返回结果，一定支持序列化
PO：用于持久化对象，与数据库交互的模型 

Model-> DO ->PO 
				   ->DTO
				 
在整个业务流程中，各个对象不要求完全一致，比如Id在PO展示就可以，没有必要放在DO中

```

# 三、Java8

## 1、default

```
在接口中添加default表示的方法。可以保证在拓展接口的时候，不影响其实现类。并且实现了可以调用其default方法。
```

```java
public interface Interfaces {
    void function(String name);
    default void function1() {
        System.out.println("true = " + true);
    }
}


public class InterfacesImpl  implements Interfaces{
    @Override
    public void function(String name) {
        System.out.println("name = " + name);
        System.out.println("调用default()方法");
      // 接口定义的default方法
        function1();
    }

    public static void main(String[] args) {
        final InterfacesImpl interfaces = new InterfacesImpl();
        interfaces.function("xiao");
    }
}


冲突的时候遵循原则：
超类优先原则：当某个类继承超类并实现接口时，如果超类和接口中都有一个相同的方法名时，如果接口设置了默认的方法实现体，那么接口的方法会被忽略，直接使用类的方法。
如果接口的方法与类的父类的方法重名时，调用的时候优先使用本类实现的方法。
  
  
1、类中的方法优先级最高，类和父类中声明的方法优先级高于任何声明为默认方法的优先级；【本类的方法最强】
2、类中无任何声明，子接口的优先级更高，高于任何声明为默认方法的优先级；
3、以上不满足，继承了多接口的类必须通过显示覆盖和调用期望的方法，显式的选择哪种默认方法；
```

## 2、计时

###  a、通过计算 

```java
// 注：仅适用于Spring
StopWatch stopWatch = new StopWatch();
stopWatch.start();
// dosomething

stopWatch.stop();
long totalTimeMillis = stopWatch.getTotalTimeMillis();
```

### b、 AOP

## 3、处理时间API

```
常用类有LocalDate、LocalTime 、LocalDateTime、DateTimeFormatter、Duration、Period
LocalDate:处理日期
LocalTime：处理时间
LocalDateTime：时间日期
DateTimeFormatter：格式化
*  LocalDate date = LocalDate.now();
*  String text = date.format(formatter);
*  LocalDate parsedDate = LocalDate.parse(text, formatter);
Duration ：处理毫秒
Period：处理年代



DateTimeFormatter dateTimeFormatter1 = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
DateTimeFormatter dateTimeFormatter2 = DateTimeFormatter.ofPattern("yyyy-MM-dd");

LocalDateTime localDateTime = LocalDateTime.parse("2019-07-31 00:00:00",dateTimeFormatter1);
LocalDate localDate = LocalDate.parse("2019-07-31",dateTimeFormatter2);
Date date = Date.from(LocalDateTime.parse("2019-07-31 00:00:00",dateTimeFormatter1).atZone(ZoneId.systemDefault()).toInstant());


String strDateTime = "2019-07-31 00:00:00";
String strDate = "2019-07-31";
Long timestamp=1564502400000l;

/** LocalDateTime 转 LocalDate */
System.out.println("LocalDateTime 转 LocalDate: "+localDateTime.toLocalDate());
/** LocalDateTime 转 Long */
System.out.println("LocalDateTime 转 Long: "+localDateTime.atZone(ZoneId.systemDefault()).toInstant().toEpochMilli());
/** LocalDateTime 转 Date */
System.out.println("LocalDateTime 转 Date: "+Date.from(localDateTime.atZone(ZoneId.systemDefault()).toInstant()));
/** LocalDateTime 转 String */
System.out.println("LocalDateTime 转 String: "+localDateTime.format(dateTimeFormatter1));

System.out.println("-------------------------------");

/** LocalDate 转 LocalDateTime */
System.out.println("LocalDate 转 LocalDateTime: "+LocalDateTime.of(localDate,LocalTime.parse("00:00:00")));
/** LocalDate 转 Long */
System.out.println("LocalDate 转 Long: "+localDate.atStartOfDay(ZoneId.systemDefault()).toInstant().toEpochMilli());
/** LocalDate 转 Date */
System.out.println("LocalDate 转 Date: "+Date.from(localDate.atStartOfDay().atZone(ZoneId.systemDefault()).toInstant()));
/** LocalDate 转 String */
System.out.println("LocalDateTime 转 String: "+localDateTime.format(dateTimeFormatter2));

System.out.println("-------------------------------");

/** Date 转 LocalDateTime */
System.out.println("Date 转 LocalDateTime: "+LocalDateTime.ofInstant(date.toInstant(), ZoneId.systemDefault()));
/** Date 转 Long */
System.out.println("Date 转 Long: "+date.getTime());
/** Date 转 LocalDate */
System.out.println("Date 转 LocalDateTime: "+LocalDateTime.ofInstant(date.toInstant(), ZoneId.systemDefault()).toLocalDate());
/** Date 转 String */
SimpleDateFormat sdf =new SimpleDateFormat("yyyy-MM-dd HH:mm:ss SSS" );
System.out.println("Date 转 String: "+sdf.format(date));

System.out.println("-------------------------------");

/** String 转 LocalDateTime */
System.out.println("String 转 LocalDateTime: "+LocalDateTime.parse(strDateTime,dateTimeFormatter1));
/** String 转 LocalDate */
System.out.println("String 转 LocalDate: "+LocalDateTime.parse(strDateTime,dateTimeFormatter1).toLocalDate());
System.out.println("String 转 LocalDate: "+LocalDate.parse(strDate,dateTimeFormatter2));
/** String 转 Date */
System.out.println("String 转 Date: "+Date.from(LocalDateTime.parse(strDateTime,dateTimeFormatter1).atZone(ZoneId.systemDefault()).toInstant()));

System.out.println("-------------------------------");

/** Long 转 LocalDateTime */
System.out.println("Long 转 LocalDateTime:"+LocalDateTime.ofInstant(Instant.ofEpochMilli(timestamp), ZoneId.systemDefault()));
/** Long 转 LocalDate */
System.out.println("Long 转 LocalDate:"+LocalDateTime.ofInstant(Instant.ofEpochMilli(timestamp), ZoneId.systemDefault()).toLocalDate());
```

## 4、Lambda

```java
ambda表达式使⽤前提:⼀个接口中只包含一个抽象方法，则可以使用Lambda表达式，也称之为“函数接口”   
语法: (params) -> expression

Lambda 表达式的实现方式在本质是以匿名内部类的方式进⾏实现
```

## 5、function

```
Consumer<T> : 消费型接口:一个入参，⽆无返回值
void accept(T t);
//定义
				Consumer<String> consumer = p-> System.out.println(p+" world!");
				//调用
        consumer.accept("Hello");

Supplier<T> : 供给型接口:⽆入参，有返回值
T get();


Function<T, R> : 函数型接⼝:一个入参，一个返回值
R apply(T t);


Predicate<T> : 断⾔型接口:一个入参，返回值类型确定是boolean
boolean test(T t);
```

##  6、Sream

```
Stream<T> filter(Predicate<? super T> predicate);
	这个接口传入一个泛型参数T，做完操作之后，返回一个boolean值；filter方法的作用，是对这个boolean做判断，返回true判断之后的对象
	返回符合条件的对象
```

## 7、String

```
将list中的对象通过特定的字符分割

String.join(",", updateEmployeeIdList)
```





