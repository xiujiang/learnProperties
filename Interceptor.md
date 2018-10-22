# Interceptor

## 1.AuthInterceptor

​	`HandlerInterceptor`

​	继承自`HandlerInterceptor`，SpringMVC 的处理器拦截器，类似于Servlet开发中的过滤器Filter,用于对请求进行拦截和处理。

​	`HandlerInterceptor` 是接口，需要实现类全部实现，`HandlerInterceptorAdapter` 是抽象类，实现了`handlerInterceptor`的全部功能，体现了 **适配器设计模式** ，我们只需要继承`HandlerInterceptorAdapter` ，重写某些方法就可以。

执行顺序
1、单个实现类的执行顺序

preHandler -> Controller -> postHandler -> model渲染-> afterCompletion

2、多个实现类的执行顺序

———————preHandler1——————- 
———————preHandler2——————- 
———————preHandler3——————- 
———————–Controller——————— 
———————postHandler3—————— 
———————postHandler2—————— 
———————postHandler1—————— 
———————postHandler1—————— 
——————afterCompletion3—————- 
——————afterCompletion2—————- 







**问题** ：1.Interceptor 是如何注册的







# WebMvcConfigurer

