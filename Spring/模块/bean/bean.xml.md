​	一、使用配置文件方试注入bean

1、先创建bean.xml 配置文件

```xml
<?xml version="1.0" encoding="utf-8" ?>

<beans xmlns="http://www.springframework.org/schema/beans"      默认命名空间
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"    xsi命名空间
       xmlns:aop="http://www.springframework.org/schema/aop"    aop:自定的命名空间，后面是命名空间全称，并必须在xsi空间中指定schemaLocation
       xsi:schemaLocation="http://www.springframework.org/schema/beans    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/beans/spring-aop-3.0.xsd>   为每个命名空间指定具体的schemaLocation,
  
  
  
    <bean id="person" class="com.huochai.demo.Person">
        <property name="age" value="12"/>
        <property name="name" value="lizhi"/>
    </bean>

</beans>


①默认命名空间：它没有空间名，用于Spring Bean的定义；

②xsi命名空间：这个命名空间用于为每个文档中命名空间指定相应的Schema样式文件，是标准组织定义的标准命名空间；

③aop命名空间：这个命名空间是Spring配置AOP的命名空间，是用户自定义的命名空间。

 
```

2、获取bean

```java
public static void main(String[] args) {
        // 解析xml 文件的对象,返回IOC容器
         ClassPathXmlApplicationContext classPathXmlApplicationContext = new ClassPathXmlApplicationContext("bean.xml");
        final Person person = (Person) classPathXmlApplicationContext.getBean("person");
        System.out.println("person = " + person);

    }
```

