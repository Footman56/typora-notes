# GET 请求

常用于查询，不做复杂的业务，一般设置参数都设置为param ，会在浏览器url 中显示参数以及对应的参数值

## 1、 请求参数置在param  中

![image-20230109141357268](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202301091413394.png)

```java
 /**
     * http://localhost:8080/test/requestLine?data=1234
     * 将data 放到body里面不行哈
     */
    @GetMapping(path = "/requestLine")
    public void testLine(String data) {
        System.out.println("data = " + data);
    }
```

注：如果前端将参数放在data里面的话，上面那种方式是获取不到参数信息的  

## 2、请求参数在body中

![image-20230109141443699](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202301091414733.png)

```java
// get 请求也可以将参数放在body 中，但是想要获取的话就需要使用@RequestBody来接受
@GetMapping(path = "/requestLine")
    public void testLine(@RequestBody String data) {
      // data = {"data":"yyyyyyyyy"} 是json 格式，想要获取参数的话就需要反序列化 
        System.out.println("data = " + data);
  }
```

# POST 请求

常用于保存数据，一般设置请求参数为body，更加安全，不会在url 中显示

## 1、请求参数置在param  中

```java
   @PostMapping(path = "/requestLine")
    public void testLinePost(String data) {
        System.out.println("data = " + data);
}
```

![image-20230109142632761](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202301091426869.png)

能获取到结果

## 2、请求参数在body中

```java
@PostMapping(path = "/requestLine")
public void testLinePost(@RequestBody String data) {
      // data = 333333
      System.out.println("data = " + data);
}
```

![image-20230109142840519](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202301091428554.png)

将参数放在body中，通过form形式传递参数

## 3、请求参数在路径上

```java
/**
     * http://localhost:8080/test/requestPath/12?data=907
     */
    @PostMapping(path = "/requestPath/{id}")
    public void testPathPost(@PathVariable(value = "id") String id, String data) {
      // id = 12
        System.out.println("id = " + id);
      // data = 907
        System.out.println("data = " + data);
    }
```

![image-20230109143812560](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202301091438620.png)

## 4、 复杂参数

参数在param 中 

```java
 /**
     * http://localhost:8080/test/requestObject?name=xx&useWay=write&writeStyle=black
     * http://localhost:8080/test/requestObject Body里面form-data 形式的参数也可以(仅限对象)
     * 需要pen 有默认构造函数
     */
    @PostMapping(path = "requestObject")
    public void testObject(Pen pen) {
        System.out.println("JsonUtil.toJson(Pen) = " + JsonUtil.toJson(pen));
}
```

![image-20230109144322922](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202301091443006.png)

可以显示出Pen 对象， 并且根据请求中参数封装到Pen对象里面



参数在body中

```java
@PostMapping(path = "requestObject")
    public void testObjectPost(@RequestBody Pen pen) {
        System.out.println("JsonUtil.toJson(Pen) = " + JsonUtil.toJson(pen));
    }
```

![image-20230109144708970](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202301091447037.png)

**注：此时一定要指定Content-Type，否则的话会提示415**

![image-20230109144955497](/Users/peilizhi/Library/Application Support/typora-user-images/image-20230109144955497.png)

# 常用注解

### 2.1 @RequestMapping  地址映射

```
value：     指定请求的实际地址，指定的地址可以是URI Template 模式（后面将会说明）；

method：  指定请求的method类型， GET、POST、PUT、DELETE等；


consumes： 指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;控制请求参数的格式 

produces:    指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回；

params： 指定request中必须包含某些参数值是，才让该方法处理。 

headers： 指定request中必须包含某些指定的header值，才能让该方法处理请求。

控制地址映射
```

### 2.2 参数获取

```
A、处理requet uri 部分（这里指uri template中variable，不含queryString部分）的注解：   @PathVariable;
B、处理request header部分的注解：   @RequestHeader, @CookieValue;
C、处理request body部分的注解：@RequestParam,  @RequestBody;
D、处理attribute类型是注解： @SessionAttributes, @ModelAttribute;
```

#### 2.2.1 @PathVariable

```java
用于获取url 中的参数。 
someUrl/{paramId}, 这时的paramId可通过 @Pathvariable注解绑定它传过来的值到方法的参数上。
  
  
@Controller
@RequestMapping("/owners/{ownerId}")
public class RelativePathUriTemplateController {
 
  @RequestMapping("/pets/{petId}")
  public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {    
    // implementation omitted
  }
}
若方法参数名称和需要绑定的uri template中变量名称不一致，需要在@PathVariable("name")指定uri template中的名称。
```

#### 2.2.2 @RequestHeader

```java
@RequestHeader 注解，可以把Request请求header部分的值绑定到方法的参数上。


Host                    localhost:8080
Accept                  text/html,application/xhtml+xml,application/xml;q=0.9
Accept-Language         fr,en-gb;q=0.7,en;q=0.3
Accept-Encoding         gzip,deflate
Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive              300

@RequestMapping("/displayHeaderInfo.do")
public void displayHeaderInfo(@RequestHeader("Accept-Encoding") String encoding,
                              @RequestHeader("Keep-Alive") long keepAlive)  {
 
  //...
 
}
```

#### 2.2.3 @CookieValue

````java
@CookieValue 可以把Request header中关于cookie的值绑定到方法的参数上。

@RequestMapping("/displayHeaderInfo.do")
public void displayHeaderInfo(@CookieValue("JSESSIONID") String cookie)  {
 
  //...
 
}
````

#### 2.2.4 @RequestParam

```
常用来处理简单类型的绑定，通过Request.getParameter() 获取的String可直接转换为简单类型的情况（ String--> 简单类型的转换操作由ConversionService配置的转换器来完成)
```

#### 2.2.5  @RequestBody

```
该注解常用来处理Content-Type: 不是application/x-www-form-urlencoded编码的内容，例如application/json, application/xml等；


它是通过使用HandlerAdapter 配置的HttpMessageConverters来解析post data body，然后绑定到相应的bean上的。
用于获取请求体内的内容
```

#### 2.2.6 @SessionAttributes

```
该注解用来绑定HttpSession中的attribute对象的值，便于在方法中的参数里使用。
```

#### 2.2.7  @ModelAttribute

```java
用于方法上时：  通常用来在处理@RequestMapping之前，为请求绑定需要从后台查询的model；根据查询条件从数据库中获取对象，并补充到函数的参数上



@ModelAttribute
public Account addAccount(@RequestParam String number) {
    return accountManager.findAccount(number);
}

就是在调用@RequestMapping的方法之前，为request对象的model里put（“account”， Account）；
  
  
  
用在参数上的@ModelAttribute示例代码：
  
  
@RequestMapping(value="/owners/{ownerId}/pets/{petId}/edit", method = RequestMethod.POST)
public String processSubmit(@ModelAttribute Pet pet) {
  
}
查询 @SessionAttributes有无绑定的Pet对象，若没有则查询@ModelAttribute方法层面上是否绑定了Pet对象，若没有则将URI template中的值按对应的名称绑定到Pet对象的各属性上。
  
查询ModelAttribute方法层面就是从 注解到方法上的对象获取。
```

#### 2.2.8  无注解

```
若要绑定的对象时简单类型：  调用@RequestParam来处理的。 
若要绑定的对象时复杂类型：  调用@ModelAttribute来处理的。
```



