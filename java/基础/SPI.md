# 一、用法

1. 创建固定 Resource/META-INF/services  目录

2. 在目录下添加接口路径

   ![image-20241016000810703](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410160008766.png)

必须为接口路径，否则识别不了子类



3. 调用接口方法

   ```java
   /**
        * 测试SPI
        * @param args
        */
       public static void main(String[] args) {
           ServiceLoader<TestSPIService> serviceLoader = ServiceLoader.load(TestSPIService.class);
           for (TestSPIService service : serviceLoader) {
               service.sayHello();
           }
       }
   ```

   