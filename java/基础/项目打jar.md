1. 

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410152242352.png" alt="image-20241015224158737" style="zoom:50%;" />

2. <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410152242845.png" alt="image-20241015224243796" style="zoom:50%;" />

3. <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410152243249.png" alt="image-20241015224315205" style="zoom:50%;" />

   main 可以不填

4. <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410152244145.png" alt="image-20241015224417107" style="zoom:50%;" />

选择输出路径

5. <img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410152245463.png" alt="image-20241015224505432" style="zoom:50%;" />
6. 

<img src="https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410152245007.png" alt="image-20241015224532965" style="zoom:50%;" />

7. 最终就有jar包

   ![](https://gcore.jsdelivr.net/gh/Footman56/imageBeds/202410152246926.png)





项目中想要引入jar里面的方法

```java
/**
     * 加载jar中的类，并且调用对应的方法
     * @param args
     */
    public static void main(String[] args) throws Exception {
        // 读取jar路径
        URL jarPath = new URL("file:/Users/peilizhi/javaProjects/classsloadTest/out/artifacts/classLoadJar_jar/classLoadJar.jar");
        URLClassLoader urlClassLoader = new URLClassLoader(new URL[]{jarPath});
        // 加载指定的类
        // 此时有个问题：就是如果加载的类存在当前classpath中的话，实际获取到的类就是当前classpath中的类，而不是从jar包中加载的
        // 因为类加载器会缓存住加载过的类，不会重新加载
        Class<?> clazz = urlClassLoader.loadClass("com.huochai.SalaryService");
        if (null != clazz) {
            Object object = clazz.newInstance();
            Double calculateSalary = (Double) clazz.getMethod("calculateSalary", Double.class).invoke(object, 1000D);
            System.out.println("calculateSalary = " + calculateSalary);
        }
    }
```

