# 一、pom.xml

```xml
<!--定时任务-->
        <dependency>
            <groupId>org.awaitility</groupId>
            <artifactId>awaitility</artifactId>
            <version>3.1.2</version>
            <scope>test</scope>
        </dependency>
```

# 二、代码

```java
/**
 * @Author peilizhi
 * @Date 2021/9/10 13:00
 **/
@Component
public class ScheduledTasks {

    private static final Logger log = LoggerFactory.getLogger(ScheduledTasks.class);

    private static final SimpleDateFormat DATE_FORMAT = new SimpleDateFormat("HH:mm:ss");

    /**
     * 可以使用cron表达式
     * 指定周期性执行
     */
    @Scheduled(fixedRate = 5000)
    public void reportCurrentTime() {
        log.info("The time is now {}", DATE_FORMAT.format(new Date()));
    }
}




@SpringBootApplication
// 开启定时任务
@EnableScheduling
@MapperScan(basePackages = "com.huochai.repository.mapper")
public class TestApplication {
    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
    }
}
```

