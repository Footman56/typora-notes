# 一、解析工具

## 1、Jackson

```
jackson-core，核心包，提供基于"流模式"解析的相关 API，它包括 JsonPaser 和 JsonGenerator。 Jackson 内部实现正是通过高性能的流模式 API 的 JsonGenerator 和 JsonParser 来生成和解析 json。
jackson-annotations，注解包，提供标准注解功能；
jackson-databind ，数据绑定包， 提供基于"对象绑定" 解析的相关 API （ ObjectMapper ） 和"树模型" 解析的相关 API （JsonNode）；基于"对象绑定" 解析的 API 和"树模型"解析的 API 依赖基于"流模式"解析的 API。
```

依赖：

```xml
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-core</artifactId>
  <version>2.9.6</version>
</dependency>
​
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-annotations</artifactId>
  <version>2.9.6</version>
</dependency>
​
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
  <version>2.9.6</version>
</dependency>
```

实际上只需要引入jackson-databind 即可，maven会自动引入jackson-annotations、jackson-core

### ObjectMapper

```
ObjectMapper可以从字符串，流或文件中解析JSON，并创建表示已解析的JSON的Java对象。
ObjectMapper也可以从Java对象创建JSON
```

```java
// 从字符串中获取java对象 
ObjectMapper objectMapper = new ObjectMapper();

        String carJson ="{ \"brand\" : \"Mercedes\", \"doors\" : 5 }";

        try {
            Car car = objectMapper.readValue(carJson, Car.class);

            System.out.println("car brand = " + car.getBrand());
            System.out.println("car doors = " + car.getDoors());
        } catch (IOException e) {
            e.printStackTrace();
        }
```

**默认情况下，Jackson通过将JSON字段的名称与Java对象中的getter和setter方法进行匹配,需要以其他方式将JSON对象字段与Java对象字段匹配，则需要使用自定义序列化器和反序列化器，或者使用一些Jackson注解。**

```java
// 从输入流获取java对象
ObjectMapper objectMapper = new ObjectMapper();
​
String carJson =
        "{ \"brand\" : \"Mercedes\", \"doors\" : 4 }";
Reader reader = new StringReader(carJson);
​
Car car = objectMapper.readValue(reader, Car.class);
```

**可以利用objectMapper.readValue从多个数据源获取Java对象**

```java
  // 从JSON数组字符串读取对象的Java List
  String jsonArray = "[{\"brand\":\"ford\"}, {\"brand\":\"Fiat\"}]";

    ObjectMapper objectMapper = new ObjectMapper();

    List<Car> cars1 = objectMapper.readValue(jsonArray, new TypeReference<List<Car>>(){});
```

```java
//ObjectMapper还可以从JSON字符串读取Java Map。 适用于不知道对象结构的
String jsonObject = "{\"brand\":\"ford\", \"doors\":5}";

ObjectMapper objectMapper = new ObjectMapper();
Map<String, Object> jsonMap = objectMapper.readValue(jsonObject,
    new TypeReference<Map<String,Object>>(){});
```

### 配置

```
1、忽略多余字段
objectMapper.configure(
    DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
2、不允许基本类型为null（json中基本类型设置为null后，会序列化失败）
objectMapper.configure(DeserializationFeature.FAIL_ON_NULL_FOR_PRIMITIVES, true);
3、日期转换（Jackson会将java.util.Date对象序列化为其long型的值）
objectMapper.setDateFormat( new SimpleDateFormat("yyyy-MM-dd"));


```

### 自定义序列化

```java
String json = "{ \"brand\" : \"Ford\", \"doors\" : 6 }";

		SimpleModule module =
		        new SimpleModule("CarDeserializer", new Version(3, 1, 8, null, null, null));
		module.addDeserializer(Car.class, new CarDeserializer(Car.class));

		ObjectMapper mapper = new ObjectMapper();
		mapper.registerModule(module);

		Car car = mapper.readValue(json, Car.class);


public class CarDeserializer extends StdDeserializer<Car> {

    public CarDeserializer(Class<?> vc) {
        super(vc);
    }

    @Override
    public Car deserialize(JsonParser parser, DeserializationContext deserializer) throws IOException {
        Car car = new Car();
        while(!parser.isClosed()){
            JsonToken jsonToken = parser.nextToken();

            if(JsonToken.FIELD_NAME.equals(jsonToken)){
                String fieldName = parser.getCurrentName();
                System.out.println(fieldName);

                jsonToken = parser.nextToken();

              // 目的是给对象赋值
                if("brand".equals(fieldName)){
                    car.setBrand(parser.getValueAsString());
                } else if ("doors".equals(fieldName)){
                    car.setDoors(parser.getValueAsInt());
                }
            }
        }
        return car;
    }
}
```

```
将对象写入Json
ObjectMapper.writeValue()
ObjectMapper.writeValueAsBytes()   返回Byte[]
ObjectMapper.writeValueAsString()  返回String
```

```java

CarSerializer carSerializer = new CarSerializer(Car.class);
		ObjectMapper objectMapper = new ObjectMapper();

		SimpleModule module =
		        new SimpleModule("CarSerializer", new Version(2, 1, 3, null, null, null));
		module.addSerializer(Car.class, carSerializer);

		objectMapper.registerModule(module);

		Car car = new Car();
		car.setBrand("Mercedes");
		car.setDoors(5);

		String carJson = objectMapper.writeValueAsString(car);




public class CarSerializer extends StdSerializer<Car> {

    protected CarSerializer(Class<Car> t) {
        super(t);
    }

    public void serialize(Car car, JsonGenerator jsonGenerator,
                          SerializerProvider serializerProvider)
            throws IOException {

        jsonGenerator.writeStartObject();
        jsonGenerator.writeStringField("producer", car.getBrand());
        jsonGenerator.writeNumberField("doorCount", car.getDoors());
        jsonGenerator.writeEndObject();
    }
}
```



