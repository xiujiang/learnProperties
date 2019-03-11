# Spring IOC

## 总述

​	Spring IOC  容器，完成了对象的创建和依赖管理注入等。主要思想为：由IOC容器关注相互依赖对象之间的关系，用户只需关注业务逻辑本身。

​	IOC (Intversion of Control) 控制反转，是一种设计思想，IOC意味着本来由你控制去创建对象`new`，变为了由IOC容器帮你去创建对象，控制和注入。由容器控制对象的生命周期和对象之间的关系。

​	DI (Dependency Injection) 依赖注入，实际上这也就是IOC的实现方式,组件之间的依赖关系是由容器运行期间决定的，即由容器动态的将某个依赖关系注入到组件中，依赖注入的目的并非为软件系统带来更多功能，而是为了提升组件重用的频率，为系统搭建灵活，可扩展的平台。



## 1.Bean注册

​	如果想要将bean交由spring容器管理，就需要首先将bean注册在spring容器中，而bean可以通过xml或者注解的方式进行注册。

​	注册方式：

​	1.注解注册Annotation

​		方式1：`@Configuration`,`@Bean` 

​		被`@Configuration`注解标识的类自动获得`@Component`的特性，因为该注解本身也是使用了`@Component`注解，具体可以查看`@Configuration`的源码定义，并且该类会作为spring的一个配置类，在创建该类型的`bean`时，spring会扫描当中所有`@Bean`注解标注的方法，并自动执行，返回值自动注册在容器中，**默认使用方法名作为bean的name**。也可以通过提供`@Bean`的**value值**或设置bean的**name属性**来给bean起名字。

​		方式2：使用`@ComponentScan`注解自动注册

​		对相关包配置扫描路径，自动创建所有标有`@Component`相关注解的类的实例。并将其注入到Spring容器中。如果是`@configuration`,还会执行内部的`@Bean`方法。

​	(`Repository`,`Controller`,`Service`,`@Configuration`)这些在内部都是被`@Component` 注释。

​		方式3: 使用`@import`注解注入Spring容器（Spring Boot 中`@EnableAutoConfiguration`的核心实现，便是`@impot`）。

 	2. XML 注入
      	1. 基于xml的配置一般是通过`<bean>`、`<context:component-scan>`等xml标签进行配置，然后由spring容器扫描xml文件进行注册

## 2.Bean注入

​	什么叫注入？ 

​	当Spring在运行期间，当某个组件想使用某个Bean时，由SpringIOC 容器通过几种方式，将Bean的实例注入到组件中，组件便可以直接使用，Bean实例是由IOC容器进行管理的。

​	三种注入方式：

​	1.构造器注入（）

​		构造器的注入，主要依赖于构造器方法的实现，构造方法可以是无参的，也可以是有参的。同样Spring也可以采用反射的方式，通过构造器来完成注入。这也是构造器注入的原理。

​	2.set方法注入

​		利用java Bean规范所定义的get/set方法完成注入。

​	3.基于注解的注入

​		这种注入方式由`Autowire`,`Resource` 注解来实现，通过配置属性，可设置构造器注入或者name,type来确定注入方式。

## 3.Bean生命周期&作用域

​		![](D:\note\SpringIocBean.png)

Bean的生命周期流程图：

![](D:\note\SpringIocBeanLife.png)

Bean的作用域：

​	![](D:\note\SpringBeanScope.png)

































