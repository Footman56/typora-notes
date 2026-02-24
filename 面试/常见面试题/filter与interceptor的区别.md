# 一、来源不同

filter 来源servlet ，Interceptor 来源spring 框架

# 二、触发时机不一样

请求进入容器 ->   filter ->   servlet  ->  interceptor -> controller

# 三、实现方式不同

filter 是基于方法回调实现的，都要执行doFilter 才能后续执行的

interceptor是基于动态代理的 