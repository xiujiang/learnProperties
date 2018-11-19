# Spring Mvc 请求流程

## 1. 请求流程

SpringMVC框架是一个基于请求驱动的Web框架，并且使用了‘前端控制器’模型来进行设计，再根据‘请求映射规则’分发给相应的页面控制器进行处理。

​	![img](https://images2015.cnblogs.com/blog/791227/201611/791227-20161125140123503-1552603846.png)



具体步骤：

1、  首先用户发送请求到前端控制器，前端控制器根据请求信息（如 URL）来决定选择哪一个页面控制器进行处理并把请求委托给它，即以前的控制器的控制逻辑部分；图中的 1、2 步骤；

2、  页面控制器接收到请求后，进行功能处理，首先需要收集和绑定请求参数到一个对象，这个对象在 Spring Web MVC 中叫命令对象，并进行验证，然后将命令对象委托给业务对象进行处理；处理完毕后返回一个 ModelAndView（模型数据和逻辑视图名）；图中的 3、4、5 步骤；

3、  前端控制器收回控制权，然后根据返回的逻辑视图名，选择相应的视图进行渲染，并把模型数据传入以便视图渲染；图中的步骤 6、7；

4、  前端控制器再次收回控制权，将响应返回给用户，图中的步骤 8；至此整个结束。



![img](https://images2015.cnblogs.com/blog/791227/201611/791227-20161125140338768-995727439.png)



具体步骤：

第一步：发起请求到前端控制器(DispatcherServlet)

第二步：前端控制器请求HandlerMapping查找 Handler （可以根据xml配置、注解进行查找）

第三步：处理器映射器HandlerMapping向前端控制器返回Handler，HandlerMapping会把请求映射为HandlerExecutionChain对象（包含一个Handler处理器（页面控制器）对象，多个HandlerInterceptor拦截器对象），通过这种策略模式，很容易添加新的映射策略

第四步：前端控制器调用处理器适配器去执行Handler

第五步：处理器适配器HandlerAdapter将会根据适配的结果去执行Handler

第六步：Handler执行完成给适配器返回ModelAndView

第七步：处理器适配器向前端控制器返回ModelAndView （ModelAndView是springmvc框架的一个底层对象，包括 Model和view）

第八步：前端控制器请求视图解析器去进行视图解析 （根据逻辑视图名解析成真正的视图(jsp)），通过这种策略很容易更换其他视图技术，只需要更改视图解析器即可

第九步：视图解析器向前端控制器返回View

第十步：前端控制器进行视图渲染 （视图渲染将模型数据(在ModelAndView对象中)填充到request域）

第十一步：前端控制器向用户响应结果



## 2.源码解析

​	按照请求的流转路径，来学习源码。

### 1.类信息

| 类名                    | extends&implements | 描述                                                       |
| ----------------------- | ------------------ | ---------------------------------------------------------- |
| DispatcherServlet       | FrameworkServlet   | 核心类，请求的主要处理者，负责协调和组织不同组件完成处理。 |
| HandlerMapping          | **interface**      | HandlerMapping的本质就是找到Controller                     |
| HandlerAdapter          |                    | 具体调用controller的方法                                   |
| ListableBeanFactory     | BeanFactory        | 提供容器中bean的迭代功能，获取所有bean,根据类型获取bean    |
| HierarchicalBeanFactory | BeanFactory        | 提供父容器的访问功能                                       |
| ViewResolver            | **interface**      | 视图解析器                                                 |
| ModelAndView            |                    | 模型视图信息                                               |
| HandlerInterceptor      | **interface**      | 拦截器顶级父类，通用为:HandlerInterceptorAdapter           |
| HandlerExecutionChain   |                    | handler执行链                                              |
| LocaleResolver          |                    | 视图解析器                                                 |
| ThemeResolver           |                    | 视图解析器                                                 |

### 2.请求处理流程

​	在整个Spring mvc 框架中，DispatcherServlet 处于核心位置，它负责协调和组织不同组件完成请求处理并返回响应工作。在看DispatcherServlet类之前，先看一下请求处理的**大致流程**：

​	1.Tomcat启动，对DispatcherServlet进行实例化，然后调用它的`init()`方法进行初始化，在这个初始化过程中完成了：

​	**对web.xml中初始化参数的加载**；

​	**建立webApplicationContext(SpringMVC的IOC容器);**

​	**进行组件的初始化。**

​	2.客户端发出请求，由Tomcat接收到这个请求，如果匹配了DispatcherServlet在web.xml中配置的映射路径，Tomcat就将请求转交给DispatcherServlet处理；

​	3.DispatcherServlet从容器中取出所有Handlermapping实例（每个实例对应一个HandlerMapping接口的实现类）并遍历，每个HandlerMapping会根据请求信息，通过自己实现类中的方式去找到处理该请求的Handler(执行程序，如Controller中的方法)，并且将这个Handler与一堆HandlerInterceptor（拦截器）封装成一个HandlerExecutionChain对象，一旦有一个HandlerMapping可以找到Handler，则退出循环；

​	4.DispatcherServlet取出HandlerAdapter组件，根据已经找到的Handler，再从所有HandlerAdapter中找到可以处理该Handler的HandlerAdapter对象；

​	5.执行HandlerExecutionChain中所有拦截器的preHandler()方法，然后再利用HandlerAdapter执行Handler，执行完成得到ModelAndView，再依次调用拦截器的postHandler()方法；

​	6.利用ViewResolver将ModelAndView或是Exception（可解析成ModelAndView）解析成View，然后View会调用render()方法再根据ModelAndView中的数据渲染出页面；	

​	7.最后再依次调用拦截器的afterCompletion()方法，请求结束。

​	

DispatcherServlet 继承自 HttpServlet，它遵循 Servlet 里的“init-service-destroy”三个阶段，首先我们先来看一下它的 init() 阶段。

#### 1.1 init()

​	DispatcherServlet --->FrameworkServlet --->HttpServletBean

​			![1541149275167](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1541149275167.png)	

Spring 容器在启动过程中，会注册ApplicationEvent   (FrameworkServlet 中的事件)			![1541149238321](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1541149238321.png)

当FrameWorkServlet 中的onRefresh 事件发生后，会初始化DispatcherServlet. 初始化中，分别会调用：

​	**1.initMultipartResolver()**：文件上传解析，如果请求类型是multipart将通过MultipartResolver进行文件上传解析 。

​	**2.initLocaleResolver()**：初始化一些多语言实现相关的类，在配置多语言本地化时会注入bean名称为localeResolver，默认实现的类有FixedLocaleResolver ，SessionLocaleResolver ，CookieLocaleResolver， AcceptHeaderLocaleResolver

​	**3.initthemeResolver()**：与样式相关的解析器，需要在配置文件中注入bean名称为themeResolver的，FixedThemeResolver, SessionThemeResolver和CookieThemeResolver。

​	**4.initHandlerMappings()**：初始化HandlerMappings，可以为每个请求找到合适的处理器。

![1541149876140](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1541149876140.png)

​	**5.initHandlerAdapters**：初始化handler适配器，handlerAdapter是真正调用Controller操作的类。

​	**6.initHandlerExceptionResolvers()**：初始化handler异常解析器，如果在执行过程中遇到异常，是由HandlerExceptionResolver来解析的。

​	**7.initRequestToViewNameTranslator()**：是初始化到ViewName的处理器，可以解析请求到视图名

​	**8.initViewResolvers()**：初始化视图解析器

​	**9.initFlashMapManager()**：初始化flash映射管理器,与链接跳转相关的。

这些在初始化的时候，会调用到org.springFrameWork.web.servlet包下的DispatcherServlet.properties的配置文件，这个配置文件中定义了一些要初始化的类：

~~~ properties
org.springframework.web.servlet.LocaleResolver=org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver
org.springframework.web.servlet.ThemeResolver=org.springframework.web.servlet.theme.FixedThemeResolver
org.springframework.web.servlet.HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
	org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping
org.springframework.web.servlet.HandlerAdapter=org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,\
	org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,\
	org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter
org.springframework.web.servlet.HandlerExceptionResolver=org.springframework.web.servlet.mvc.method.annotation.ExceptionHandlerExceptionResolver,\
	org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver,\
	org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver
org.springframework.web.servlet.RequestToViewNameTranslator=org.springframework.web.servlet.view.DefaultRequestToViewNameTranslator
org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver
org.springframework.web.servlet.FlashMapManager=org.springframework.web.servlet.support.SessionFlashMapManager
~~~

#### 1.2 DispatcherServlet

​	DispatcherServlet 继承自HttpServlet，所以它和普通的HttpServlet有同样的配置。

```xml
<servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>
            org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
	    <init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:springmvc-config.xml</param-value>
		</init-param>
        <load-on-startup>1</load-on-startup>    
	</servlet>
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>*.action</url-pattern>
```

​	在这个servlet配置中，DispatcherServlet会被所有的*.action请求所调用。因为它是一个HttpServlet，所以它也实现了HttpServlet提供的方法，都在DispatcherServlet的父类`FrameworkServlet`中实现。

​	其默认实现为：

```java
@Override
protected final void doPost(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {
	processRequest(request, response);}
@Override
protected final void doGet(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {
	processRequest(request, response);
}
```

processRequest的实现是在FrameworkServlet中，在这个方法中，最主要的操作就是调用doServlet方法

![1541152474871](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1541152474871.png)

doService 方法最终实现是在DispatcherServlet中，这样所有的http请求（get,post,put,delete）的最终操作就是DispatcherServlet实现了。

​	DispatcherServlet中doService的实现如下，对Request设置了一些全局属性，最终接下来的操作是在doDispatcher函数中实现了。

```java
	//获取请求，设置一些request的参数，然后分发给doDispatch
	@Override
protected void doService(HttpServletRequest request, HttpServletResponse response) 	throws Exception {
	if (logger.isDebugEnabled()) {
			String resumed = WebAsyncUtils.getAsyncManager(request).hasConcurrentResult() ? " resumed" : "";
			logger.debug("DispatcherServlet with name '" + getServletName() + "'" + resumed +" processing " + request.getMethod() + " request for [" + getRequestUri(request) + "]");
		}
  // Keep a snapshot of the request attributes in case of an include,
	// to be able to restore the original attributes after the include.
	Map<String, Object> attributesSnapshot = null;
	if (WebUtils.isIncludeRequest(request)) {
		attributesSnapshot = new HashMap<String, Object>();
		Enumeration<?> attrNames = request.getAttributeNames();
		while (attrNames.hasMoreElements()) {
			String attrName = (String) attrNames.nextElement();
			if (this.cleanupAfterInclude || attrName.startsWith("org.springframework.web.servlet")) {
				attributesSnapshot.put(attrName, request.getAttribute(attrName));
			}
		}
	}
 
	// Make framework objects available to handlers and view objects.
	/* 设置web应用上下文**/
	request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
	/* 国际化本地**/
	request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
	/* 样式**/
	request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
	//设置样式资源
	request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());
	//请求刷新时保存属性
	FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
	if (inputFlashMap != null) {
		request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
	}
	//Flash attributes 在对请求的重定向生效之前被临时存储（通常是在session)中，并且在重定向之后被立即移除
	request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
	//FlashMap 被用来管理 flash attributes 而 FlashMapManager 则被用来存储，获取和管理 FlashMap 实体.
	request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
 
	try {
		doDispatch(request, response);
	}
	finally {
		if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
			// Restore the original attribute snapshot, in case of an include.
			if (attributesSnapshot != null) {
				restoreAttributesAfterInclude(request, attributesSnapshot);
			}
		}
	}}
```

​	doDispatch函数中完成了对一个请求的所有操作。

```java
/**
	 *将Handler进行分发，handler会被handlerMapping有序的获得
	 *通过查询servlet安装的HandlerAdapters来获得HandlerAdapters来查找第一个支持handler的类
	 *所有的HTTP的方法都会被这个方法掌控。取决于HandlerAdapters 或者handlers 他们自己去决定哪些方法是可用
	 *@param request current HTTP request
	 *@param response current HTTP response
	 */
	protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		/* 当前HTTP请求**/
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;
	WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
 
try {
	ModelAndView mv = null;
	Exception dispatchException = null;
	
try {
	//判断是否有文件上传
	processedRequest = checkMultipart(request);
	multipartRequestParsed = (processedRequest != request);
	
	// 获得HandlerExecutionChain，其包含HandlerIntercrptor和HandlerMethod
	mappedHandler = getHandler(processedRequest);
	if (mappedHandler == null || mappedHandler.getHandler() == null) {
		noHandlerFound(processedRequest, response);
		return;
	}			
	//获得HandlerAdapter
	HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
	
	//获得HTTP请求方法
	String method = request.getMethod();
	boolean isGet = "GET".equals(method);
if (isGet || "HEAD".equals(method)) {
	long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
		if (logger.isDebugEnabled()) {
			logger.debug("Last-Modified value for [" + getRequestUri(request) + "] is: " + lastModified);
	}
if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
			return;
}
}
	//如果有拦截器的话，会执行拦截器的preHandler方法
	if (!mappedHandler.applyPreHandle(processedRequest, response)) {
		return;
	}
 
	//返回ModelAndView
	mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
 
	if (asyncManager.isConcurrentHandlingStarted()) {
		return;
	}
	//当view为空时，，根据request设置默认view
	applyDefaultViewName(processedRequest, mv);
	//执行拦截器的postHandle
	mappedHandler.applyPostHandle(processedRequest, response, mv);
		}
		catch (Exception ex) {
			dispatchException = ex;
		}
		processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
	}
	catch (Exception ex) {
		triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
	}
	catch (Error err) {
		triggerAfterCompletionWithError(processedRequest, response, mappedHandler, err);
	}
	finally {
		//判断是否是异步请求
		if (asyncManager.isConcurrentHandlingStarted()) {
			// Instead of postHandle and afterCompletion
			if (mappedHandler != null) {
				mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
			}
		}
		else {
			// Clean up any resources used by a multipart request.
			//删除上传资源
			if (multipartRequestParsed) {
				cleanupMultipart(processedRequest);
			}
		}
    }
```

调用完doDispatch之后就完成了一个请求的访问，其会将渲染后的页面或者数据返回给请求发起者。

#### 1.3 HandlerMapping

​	我们知道，在对DispatcherServlet做初始化之前，WebApplicationContext已经加载完成，IOC容器也已经工作。在初始化DispatcherServlet的时候会加载HandlerMapping,  initHandlerMappings核心方法为：

​	**BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);**

```java
private void initHandlerMappings(ApplicationContext context) {
		this.handlerMappings = null;

		if (this.detectAllHandlerMappings) {
			// Find all HandlerMappings in the ApplicationContext, including ancestor contexts.
			Map<String, HandlerMapping> matchingBeans =
					BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
			if (!matchingBeans.isEmpty()) {
				this.handlerMappings = new ArrayList<HandlerMapping>(matchingBeans.values());
				// We keep HandlerMappings in sorted order.
				AnnotationAwareOrderComparator.sort(this.handlerMappings);
			}
            ……………………
		}
```

这个方法中，首先通过lbf也就是WebApplicationContext来获取所有HandlerMapping类型的的bean。如果类型为HierarchicalBeanFactory，则把父类ListableBeanFactory中的bean获取到。

```java
public static <T> Map<String, T> beansOfTypeIncludingAncestors(
      ListableBeanFactory lbf, Class<T> type, boolean includeNonSingletons, boolean allowEagerInit)
      throws BeansException {

   Assert.notNull(lbf, "ListableBeanFactory must not be null");
   Map<String, T> result = new LinkedHashMap<String, T>(4);
   result.putAll(lbf.getBeansOfType(type, includeNonSingletons, allowEagerInit));
   if (lbf instanceof HierarchicalBeanFactory) {
      HierarchicalBeanFactory hbf = (HierarchicalBeanFactory) lbf;
      if (hbf.getParentBeanFactory() instanceof ListableBeanFactory) {
         Map<String, T> parentResult = beansOfTypeIncludingAncestors(
               (ListableBeanFactory) hbf.getParentBeanFactory(), type, includeNonSingletons, allowEagerInit);
         for (Map.Entry<String, T> entry : parentResult.entrySet()) {
            String beanName = entry.getKey();
            if (!result.containsKey(beanName) && !hbf.containsLocalBean(beanName)) {
               result.put(beanName, entry.getValue());
            }
         }
      }
   }
   return result;
}
```

#### 1.3 HandlerInterceptor

​	 拦截器：拦截器提供了三个方法，`preHandle`、`postHandle`、`afterCompletion`.

```java
public interface HandlerInterceptor {
    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return true;
    }
    default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {
    }
    default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {
    }
}
```

**preHandle** 方法，顾名思义，也就是在调用Controller之前调用处理，SpringMvc中可以同时存在多个Interceptor,当存在多个Interceptor时，SpringMvc会根据声明的前后顺序去调用执行，并且所有的preHandle方法都在Controller之前调用。当多个preHandle方法执行时，当某个preHandle返回false后，会中断执行，请求执行结束。

**postHandle** 方法，当preHandle方法执行结束后，调用Controller方法对请求进行处理，处理完成后，会调用postHandle方法，postHandle方法会在DispatcherServlet进行视图渲染之前调用，所以在postHandle方法中可以对ModelView进行处理。当有多个Interceptor时，调用postHandle的顺序和preHandle的正好相反，先定义的反而后执行。

**afterCompletion** 这个方法是在DispatcherServlet对视图进行渲染之后调用的。这个方法主要的作用在于清理资源。

![1542210560528](D:\note\1542210560528.png)



doDispatch方法处理Interceptor

~~~ java
	//调用HandlerExecutionChain的applyPreHandle方法
if (!mappedHandler.applyPreHandle(processedRequest, response)) {
		return;
}
//HandlerInterceptor 最终的调用是通过HandlerExecutionChain，来看一下这个类：
public class HandlerExecutionChain {
private final Object handler;	//存储HandlerMethod
private HandlerInterceptor[] interceptors;	//HandlerInterceptor数组
private List<HandlerInterceptor> interceptorList;	//所有interceptor的链表
private int interceptorIndex = -1;
    
/**
* HandlerExecutionChain
* 获取所有的HandlerInterceptor，循环进行执行，当某个preHandler执行失败，返回false
*/
 boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HandlerInterceptor[] interceptors = this.getInterceptors();
        if (!ObjectUtils.isEmpty(interceptors)) {
            for(int i = 0; i < interceptors.length; this.interceptorIndex = i++) {
                HandlerInterceptor interceptor = interceptors[i];
                if (!interceptor.preHandle(request, response, this.handler)) {
                    this.triggerAfterCompletion(request, response, (Exception)null);
                    return false;
                }
            }
        }

        return true;
    }

/**
* postHandler 获取所有的interceptor之后，倒序进行循环执行。
*/
  void applyPostHandle(HttpServletRequest request, HttpServletResponse response, @Nullable ModelAndView mv) throws Exception {
        HandlerInterceptor[] interceptors = this.getInterceptors();
        if (!ObjectUtils.isEmpty(interceptors)) {
            for(int i = interceptors.length - 1; i >= 0; --i) {
                HandlerInterceptor interceptor = interceptors[i];
                interceptor.postHandle(request, response, this.handler, mv);
            }
        }

    }
~~~

**triggerAfterCompletion(processedRequest, response, mappedHandler, ex)**：最终会调用HandlerInterceptor的afterCompletion 方法。

**triggerAfterCompletionWithError(processedRequest, response, mappedHandler, err)**：最终会调用HandlerInterceptor的afterCompletion 方法

~~~ java
void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, @Nullable Exception ex) throws Exception {
        HandlerInterceptor[] interceptors = this.getInterceptors();
        if (!ObjectUtils.isEmpty(interceptors)) {
            for(int i = this.interceptorIndex; i >= 0; --i) {
                HandlerInterceptor interceptor = interceptors[i];

                try {
                    interceptor.afterCompletion(request, response, this.handler, ex);
                } catch (Throwable var8) {
                    logger.error("HandlerInterceptor.afterCompletion threw exception", var8);
                }
            }
        }

    }
~~~

​	通过以上代码分析我们可以看到HandlerInterceptor拦截器的最终调用实现是在DispatcherServlet的doDispatch方法中，并且SpringMVC提供了HandlerExecutionChain来帮助我们执行所有配置的HandlerInterceptor拦截器，并分别调用HandlerInterceptor所提供的方法。

#### 1.4 HandlerAdapter

​	**HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());**

​	HandlerAdapter 的功能就是去执行我们的Controller，Servlet或者HttpRequestHandler中的方法。它定义了三个方法，用于处理Handler。​	

​	1.boolean supports(Object handler); 判断是否支持传入的Handler

​	2.ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)  用来使用Handler处理请求.

​	3.long getLastModified(HttpServletRequest request, Object handler); 用来获取资料的Last-Modified值。

在DispatcherServlet 初始化中，就已经注入了几个HandlerAdapter，让我们来看一下这些HandlerAdapter​	**1.SimpleServletHandlerAdapter**

​	SimpleServletHandlerAdapter 实际就是执行HttpServlet的service方法,它其实就是一个Servlet的适配器，其最终是执行Servlet的service方法，

**2.SimpleControllerHandlerAdapter**

​	SimpleControllerHandlerAdapter 实际就是执行Controller的handleRequest方法。我们来看一下这个HanderAdapter源码，在执行handle方法，直接强制转化为controller类，并且调用handleRequest();

```java
public class SimpleControllerHandlerAdapter implements HandlerAdapter {
    ………………
    @Nullable
    public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return ((Controller)handler).handleRequest(request, response);
    }
 	………………
}
```

**3.HttpRequestHandlerAdapter**

​	HttpRequestHandlerAdapter实际就是执行HttpRequestHandler的handleRequest方法。

**4.RequestMappingHandlerAdapter**

​	RequestMappingHandlerAdapter实际就是执行@RequestMapping注解的方法。

 #### 1.5 ModelAndView

​	modelAndView从这个名字中我们就可以知道这个类的大致作用，这个类包括两方面：`model`和`view`,

model 是接收到将要返回的模型信息，而view 则代表了要返回的view视图信息。来看一下源码：

```java
/** View instance or view name String */
private Object view;

/** Model Map */
private ModelMap model;

/** Optional HTTP status for the response */
private HttpStatus status;

/** Indicates whether or not this instance has been cleared with a call to {@link #clear()} */
private boolean cleared = false;
```

modelAndView 类中存放了一个view 视图或视图名称，model是一个modelMap，里面存放着我们要返回的数据信息。看构造方法，ModelMap(String,Object),接收一个String类型的attributeName,并把信息放入其中。

```java
public ModelMap(String attributeName, Object attributeValue) {
   addAttribute(attributeName, attributeValue);
}
```

![1542621818388](1542621818388.png)

在modelAndview中，构造方法由8个，String类型的作为name,在接收到后赋值到view上面，Map类型为model信息，HttpStatus代表了当前响应是否成功。

![1542621978054](1542621978054.png)

当构造好一个ModelAndView类对象的时候，DispatcherServlet会获取视图名称，如果mv视图名为空，则获取默认视图名称， 否则使用controller传回的view Name,所以，我们在Controller中，指定了返回的视图信息，则使用返回的视图。

```java
//设置ViewName
private void applyDefaultViewName(HttpServletRequest request, ModelAndView mv) throws Exception {
   if (mv != null && !mv.hasView()) {
      mv.setViewName(getDefaultViewName(request));
   }
}
```

```java

```



#### 1.6 ViewResolver

在初始化DispatcherServlet的时候就已经初始化了所有的ViewResolver,初始ViewResolvers的过程，同HandlerMapping 相同。会根据Context找到Spring 应用上下文中所有的ViewResolvers信息。

~~~ java
//获取ModelAndView之后，来执行方法。
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
      HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {
~~~

当ModelAndView获取到后，整个请求Controller也就结束，调用这个方法来判断请求是否成功，如果exception不为空，则设置errorView，然后调用这个方法渲染view。

​								**render(mv, request, response);**

在渲染中，如果view是String类型，则先根据viewName获取view视图，然后渲染view，如果正常结束，则调用Interceptor的afterCompletion方法，最后修改信息。

#### 1.7 LocalResolver

其主要作用在于根据不同的用户区域展示不同的视图，而用户的区域也称为Locale，该信息是可以由前端直接获取的。通过这种方式，可以实现一种国际化的目的，比如针对美国用户可以提供一个视图，而针对中国用户则可以提供另一个视图。在org.springframework.web.servlet.i18n这个包下，有几个Localresolver解析器。

**AcceptHeaderLocaleResolver**

​	SpringMvc 默认采用的区域解析器。通过Http请求头部解析请求。

**SessionLocalResolver**

​	它通过检验用户会话中预置的属性来解析区域。如果该会话属性不存在，它会根据accept-language HTTP头部确定默认区域。

**CookieLocaleResolver**

​	使用用户浏览器中的Cookie解析。如果Cookie不存在，它会根据accept-language HTTP头部确定默认区域。 

​	当init加载完成后，LocalResolver会注入`AcceptHeaderLocaleResolver`这个类进去。当调用render方法时，会根据reuqest调用resolverLocal方法，设置Local.

```java
protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
   // Determine locale for request and apply it to the response.
   Locale locale = this.localeResolver.resolveLocale(request);
   response.setLocale(locale);
```

```java
/** AcceptHeaderLocaleResolver中的实现  */
@Override
public Locale resolveLocale(HttpServletRequest request) {
   Locale defaultLocale = getDefaultLocale();
   if (defaultLocale != null && request.getHeader("Accept-Language") == null) {
      return defaultLocale;
   }
   Locale locale = request.getLocale();
   if (!isSupportedLocale(locale)) {
      locale = findSupportedLocale(request, locale);
   }
   return locale;
}
```

#### 1.8 ThemeResolver

​	主题就是系统的整体样式或风格，可通过Spring MVC框架提供的主题（theme）设置应用的整体样式风格，提高用户体验。Spring MVC的主题就是一些静态资源的集合，即包括样式及图片，用来控制应用的视觉风格。在Spring上下文中定义了Theme后，DispatcherServlet会在Spring容器中查找id为themeResolver的Bean并使用。ThemeResolver工作原理与LocaleResolver工作原理基本是一样的，它在request中查找theme主题并可以修改request的theme主题。

**FixedThemeResolver**

​	使用固定的主题，主题的名字（就是主题的属性文件名）可通过`defaultThemeName`属性指定，该值默认是`theme`

**CookieThemeResolver**、**SessionThemeResolver**

​	同LocalResolver。





















# Controller注册过程

































































https://blog.csdn.net/qq924862077/article/details/53523713



