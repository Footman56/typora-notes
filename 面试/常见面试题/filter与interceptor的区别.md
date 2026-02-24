# 一、来源不同

filter 来源servlet ，Interceptor 来源spring 框架

# 二、触发时机不一样

请求进入容器 ->   filter ->   servlet  ->  interceptor -> controller

# 三、实现方式不同

filter 是基于方法回调实现的，都要执行doFilter 才能后续执行的

interceptor是基于动态代理的 

# 四、作用范围不同

filter 能拦截所有web资源，Jsp，静态资源，

interceptor 拦截进入Controller 的方法调用

# 方法细粒度

Filter 只有doFilter 一个方法，不能获取Controller的名称、参数和注解，可以修改请求响应头，

Interceptor 可以获取Method 方法，目标方法参数，操作ModelAndView 对象



# 典型场景

| 场景                       | 推荐使用                      | 原因                                                         |
| :------------------------- | :---------------------------- | :----------------------------------------------------------- |
| **解决跨域（CORS）**       | **Filter**                    | 需要在 Servlet 级别提前处理 `OPTIONS` 请求，修改响应头。Spring 的 `CorsFilter` 就是基于 Filter 实现的。 |
| **防止 XSS/ SQL 注入攻击** | **Filter**                    | 通常需要包装 `HttpServletRequestWrapper`，修改请求中的参数值。 |
| **请求日志记录**           | **Interceptor** 或 **Filter** | 如果只需要记录 Controller 的执行时间，Interceptor 更合适（因为知道具体是哪个方法）。如果记录所有的请求（包括静态资源），用 Filter。 |
| **权限校验（登录检查）**   | **Interceptor**               | 更灵活，可以通过 `@PreHandle` 返回 `false` 直接阻止访问，且能结合 Spring MVC 的注解（如 `@Secured`）进行细粒度控制。 |
| **修改 ModelAndView**      | **Interceptor**               | 只有 Interceptor 可以在 `postHandle` 中直接操作 `ModelAndView`，Filter 做不到。 |
| **设置字符编码**           | **Filter**                    | Spring 的 `CharacterEncodingFilter` 就是典型，需要在读取请求体之前设置编码。 |