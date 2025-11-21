一、pom

1. 需要jdk   17
2. springboot 3. 以上
3. 配置pom文件

```xml
<properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>3.3.2</spring-boot.version>
  			<!-- Spring AI Alibaba -->
				<spring-ai-alibaba.version>1.0.0-M5.1</spring-ai-alibaba.version>
  			<!-- Spring Boot -->
				<spring-boot.version>3.4.0</spring-boot.version>
    </properties>

<dependency>
			<groupId>com.alibaba.cloud.ai</groupId>
			<artifactId>spring-ai-alibaba-starter</artifactId>
			<version>${spring-ai-alibaba.version}</version>
</dependency>
```

```xml
<repositories>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
		<repository>
			<id>aliyunmaven</id>
			<name>aliyun</name>
			<url>https://maven.aliyun.com/repository/public</url>
		</repository>
	</repositories>
```

二、获取api-key

```yaml
spring:
  application:
    name: spring-ai-alibaba-dashscope-image-example
    
  ai:
    dashscope:
      api-key: ${AI_DASHSCOPE_API_KEY}
```



三、创建controller 

1. 生成图片

```java
public class DashScopeImageController {

	private final ImageModel imageModel;

	private static final String DEFAULT_PROMPT = "为人工智能生成一张富有科技感的图片！";

	public DashScopeImageController(ImageModel imageModel) {
		this.imageModel = imageModel;
	}

	@GetMapping("/image")
	public void image(HttpServletResponse response) {

		ImageResponse imageResponse = imageModel.call(new ImagePrompt(DEFAULT_PROMPT));
		String imageUrl = imageResponse.getResult().getOutput().getUrl();

		try {
			URL url = URI.create(imageUrl).toURL();
			InputStream in = url.openStream();

			response.setHeader("Content-Type", MediaType.IMAGE_PNG_VALUE);
			response.getOutputStream().write(in.readAllBytes());
			response.getOutputStream().flush();
		} catch (IOException e) {
			response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
		}
	}

}
```
