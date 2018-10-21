# Spring Boot 启动

我们在开发spring boot 项目时，都会使用一个启动类，Spring boot的启动，是通过这个启动类进行的。

~~~ java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
~~~

这个启动类，需要我们关注的地方有两处，一个是`@SpringBootApplication`注解，另一个是`SpringApplicat ion.run(Object obj,args)` 。

### SpringBootApplication 注解

`SpringBootApplication`注解内部有三个注解。

```java
@SpringBootConfiguration	（内部为@Configuration）
@EnableAutoConfiguration
@ComponentScan
```

#### 1.@Configuration

​	这个注解是spring的注解，用来定义配置类，可替换xml配置文件，被注解的类内部包含有一个或多个被@Bean注解的方法，这些方法将会被AnnotationConfigApplicationContext或AnnotationConfigWebApplicationContext类进行扫描，并用于构建bean定义，初始化Spring容器。

**@Configuation等价于<Beans></Beans>**

**注意**：@Configuration注解的配置类有如下要求：

1. @Configuration不可以是final类型；
2. @Configuration不可以是匿名类；
3. 嵌套的configuration必须是静态类。

一、用@Configuration 加载Spring

​	1.1 @Configuration配置spring并启动spring容器

​	1.2 @Configuration启动容器+@Bean注册Bean

​	1.3 @Configuration启动容器+@Component注册Bean

​	1.4 使用AnnotationConfigApplicationContext 注册AppContext类的两种方法

​	1.5 配置Web应用程序（web.xml中配置AnnotationConfigApplicationContext）

二、组合多个配置类

​	2.1 在@configuration中引入spring的xml配置文件

​	2.2 在@configuration中引入其他注解配置

​	2.3 @configuration嵌套

三、@Enable***注解

四、@Profile逻辑组配置

五、使用外部变量

#### 2.@ComponentScan

​	ComponentScan这个注解相当于xml中`<context:component-scan base-package="\***.***.***"></context:component-scan>`	标签。

​	功能是自动扫描并加载符合条件的组件，将bean加载到IOC容器中，我们可以通过`basePackages`来细粒度定制@ComponentScan自动扫描范围。如果不指定，则默认spring框架实现会从声明@ComponentScan所在类的package进行扫描。因为这个原因，**所以我们在放置这个启动类的时候，要放在root 的package下。**

#### 3.@EnableAutoConfiguration

​	首先先了解`@Enable*`前缀注解，在这种注解里面，都包含`@Import`注解，即是导入配置类和java类

​	在这个`@EnableAutoConfiguration`注解中，最重要的是`@Import({EnableAutoConfigurationImportSelector.class})`方法。借助`EnableAutoConfigurationImportSelector`，`@EnableAutoConfiguration`可以帮助SpringBoot应用将所有符合条件的`@Configuration`配置都加载到当前SpringBoot创建并使用的IoC容器。

借助于Spring框架原有的一个工具类：SpringFactoriesLoader的支持，`@EnableAutoConfiguration`可以智能的自动配置功效才得以大功告成！

![EnableAutoConfiguration](D:\note\@EnableApplicationConfigurater.png)

在AutoConfigurationImportSelector类中可以看到通过 **SpringFactoriesLoader.loadFactoryNames() **

把**spring-boot-autoconfigure.jar/META-INF/spring.factories**中每一个xxxAutoConfiguration文件都加载到容器中，spring.factories文件里每一个xxxAutoConfiguration文件一般都会有下面的条件注解:	

@ConditionalOnClass ： classpath中存在该类时起效
@ConditionalOnMissingClass ： classpath中不存在该类时起效
@ConditionalOnBean ： DI容器中存在该类型Bean时起效
@ConditionalOnMissingBean ： DI容器中不存在该类型Bean时起效
@ConditionalOnSingleCandidate ： DI容器中该类型Bean只有一个或@Primary的只有一个时起效
@ConditionalOnExpression ： SpEL表达式结果为true时
@ConditionalOnProperty ： 参数设置或者值一致时起效
@ConditionalOnResource ： 指定的文件存在时起效
@ConditionalOnJndi ： 指定的JNDI存在时起效
@ConditionalOnJava ： 指定的Java版本存在时起效
@ConditionalOnWebApplication ： Web应用环境下起效

**SpringFactoriesLoader**

​	SpringFactoriesLoader属于Spring框架私有的一种扩展方案(类似于Java的SPI方案java.util.ServiceLoader)，其主要功能就是从指定的配置文件META-INF/spring-factories加载配置，spring-factories是一个典型的java properties文件，只不过Key和Value都是Java类型的完整类名。

### SpringApplication.run（）

#### 1.new SpringApplication()

​	调用run方法后，会调用

```java
public static ConfigurableApplicationContext run(Object[] sources, String[] args) {
   return new SpringApplication(sources).run(args);
}
```

​	首先看 new SpringApplication(Source),在这个方法内，会调用initialize(Object[] sources),初始化方法，这个方法有两个需要注意的地方。

```java
private void initialize(Object[] sources) {
   if (sources != null && sources.length > 0) {
      this.sources.addAll(Arrays.asList(sources));
   }
   this.webEnvironment = deduceWebEnvironment();
   setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
   setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
   this.mainApplicationClass = deduceMainApplicationClass();
}
```

​	deduceWebEnvironment() 这个方法会初始化webEnvironment。看其内部实现，确定了当前是以web应用启动还是以普通的jar启动,是通过classPath下是否有这两个类。

```properties
"javax.servlet.Servlet",
      "org.springframework.web.context.ConfigurableWebApplicationContext"
```

​	环境确定为web 之后，然后setInitializers方法初始化了所有的ApplicationContextInitializer，调用SpringApplication对象中的getSpringFactoriesInstances方法,来获取ApplicationContextInitializer类型对象的列表.看getSpringFactoriesInstances()方法，

```java
Set<String> names = new LinkedHashSet<String>(
      SpringFactoriesLoader.loadFactoryNames(type, classLoader));
```

​	在该方法中，首先通过调用**SpringFactoriesLoader.loadFactoryNames(type, classLoader)**来获取所有Spring Factories的名字(在包spring-boot-版本.jar下)。可以看到，是从一个名字叫spring.factories的资源文件中，读取key为org.springframework.context.ApplicationContextInitializer的value。而spring.factories的部分内容如下：

​	![1540019314260](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1540019314260.png)

![1540019381671](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1540019381671.png)

我们从上面的spring.factories的资源文件中可以看到,得到的是     **ConfigurationWarningsApplicationContextInitializer**
**ContextIdApplicationContextInitializer**
**DelegatingApplicationContextInitializer**

得到这几个name之后，会创建实例化对象，创建好实例化对象后，会sort()排序后返回list

![1540019602177](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1540019602177.png)



继续初始化Listener,步骤和`ApplicationContextInitializer`一样。会初始化

![1540019789469](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1540019789469.png)



![ApplicationListener](D:\note\ApplicationListener.png)





调用deduceMainApplicationClass（）这个方法。通过获取当前调用栈，找到入口方法main所在的类，并将其复制给SpringApplication对象的成员变量mainApplicationClass

#### 2. run()

初始化SpringApplication之后，调用run()方法，先把代码都贴出来，然后挨个分析。

```java
public ConfigurableApplicationContext run(String... args) {
   StopWatch stopWatch = new StopWatch();
   stopWatch.start();
   ConfigurableApplicationContext context = null;

   System.setProperty(
         SYSTEM_PROPERTY_JAVA_AWT_HEADLESS,
         System.getProperty(SYSTEM_PROPERTY_JAVA_AWT_HEADLESS,
               Boolean.toString(this.headless)));

   Collection<SpringApplicationRunListener> runListeners = getRunListeners(args);
   for (SpringApplicationRunListener runListener : runListeners) {
      runListener.started();
   }

   try {
      // Create and configure the environment
      ConfigurableEnvironment environment = getOrCreateEnvironment();
      configureEnvironment(environment, args);
      for (SpringApplicationRunListener runListener : runListeners) {
         runListener.environmentPrepared(environment);
      }
      if (this.showBanner) {
         printBanner(environment);
      }

      // Create, load, refresh and run the ApplicationContext
      context = createApplicationContext();
      if (this.registerShutdownHook) {
         try {
            context.registerShutdownHook();
         }
         catch (AccessControlException ex) {
            // Not allowed in some environments.
         }
      }
      context.setEnvironment(environment);
      postProcessApplicationContext(context);
      applyInitializers(context);
      for (SpringApplicationRunListener runListener : runListeners) {
         runListener.contextPrepared(context);
      }
      if (this.logStartupInfo) {
         logStartupInfo(context.getParent() == null);
      }

      // Load the sources
      Set<Object> sources = getSources();
      Assert.notEmpty(sources, "Sources must not be empty");
      load(context, sources.toArray(new Object[sources.size()]));
      for (SpringApplicationRunListener runListener : runListeners) {
         runListener.contextLoaded(context);
      }

      // Refresh the context
      refresh(context);
      afterRefresh(context, args);
      for (SpringApplicationRunListener runListener : runListeners) {
         runListener.finished(context, null);
      }

      stopWatch.stop();
      if (this.logStartupInfo) {
         new StartupInfoLogger(this.mainApplicationClass).logStarted(
               getApplicationLog(), stopWatch);
      }
      return context;
   }
   catch (Throwable ex) {
      try {
         for (SpringApplicationRunListener runListener : runListeners) {
            finishWithException(runListener, context, ex);
         }
         this.log.error("Application startup failed", ex);
      }
      finally {
         if (context != null) {
            context.close();
         }
      }
      ReflectionUtils.rethrowRuntimeException(ex);
      return context;
   }
}
```

1. 实例化StopWatch（）并且启动.StopWatch是来自org.springframework.util的工具类，可以用来方便的记录程序的运行时间。
2. 设置系统属性java.awt.headless，设置为true
3. 获取`SpringApplicationRunListener`的实例化对象`EventPublishingRunListener`，在创建和更新ApplicationContext方法前后分别调用了listeners对象的started方法和finished方法, 并在创建和刷新ApplicationContext时，将listeners作为参数传递到了createAndRefreshContext方法中，以便在创建和刷新ApplicationContext的不同阶段，调用listeners的相应方法以执行操作。所以，所谓的SpringApplicationRunListeners实际上就是在SpringApplication对象的run方法执行的不同阶段，去执行一些操作，并且这些操作是可配置的

`EventPublishingRunListener`对象在初始化时候将SpringApplication对象的成员变量listeners全都保存下来。

等到自己的public方法被调用时，发布相应的事件，或执行相应的操作。

​	RunListener是在SpringApplication对象的run方法执行到不同的阶段时，发布相应的event给SpringApplication对象的成员变量listeners中记录的事件监听器。

```java
public EventPublishingRunListener(SpringApplication application, String[] args) {
   this.application = application;
   this.args = args;
   this.multicaster = new SimpleApplicationEventMulticaster();
   for (ApplicationListener<?> listener : application.getListeners()) {
      this.multicaster.addApplicationListener(listener);
   }
}
```

​	默认情况下,调用listeners的started方法,发布了ApplicationStartedEvent时，我们已经加载的事件监听器都做了什么操作,(实现了ApplicationListener接口的类,并且事件类型是ApplicationStartingEvent类型)。

 

​	SpringApplicationRunListeners相关的类结构:

​		![](D:\note\SpringApplicationRunListeners.png)

​	4.





