---
layout: post
title: Spring Boot启动流程
category: note
tags: [spring boot]
keywords: [spring]
no-post-nav: true
---

Spring Boot 项目可通过将@SpringBootApplication标注的类作为primarySource传入SpringApplication.run(primarySource, args)方法并执行该方法的方式启动。
近日梳理了一下和启动流程相关的知识，故在此记录一下。

## Spring Boot的核心注解
@SpringBootApplication是一个组合注解，包含的主要注解包括：
- @SpringBootConfiguration，功能和@Configuration类似，标注当前类为配置类。
- @ComponentScan，自动扫描并加载符合条件的组件（比如@Component和@Repository等）或者bean定义，最终将这些bean定义加载到IoC容器中。我们可以通过basePackages等属性来细粒度的定制@ComponentScan自动扫描的范围，如果不指定，则默认Spring框架实现会从声明@ComponentScan所在类的package进行扫描。
- @EnableAutoConfiguration，通过导入@Import({AutoConfigurationImportSelector.class})，在内部通过 __SpringFactoriesLoader__ 类从classpath中搜寻所有的META-INF/spring.factories配置文件，并将其中以 _org.springframework.boot.autoconfigure.EnableAutoConfiguration_ 为key的所有配置类组合起来，再通过条件注解筛选进行自动加载配置。

## spring boot常用的条件注解
- @ConditionalOnBean，仅在当前上下文中存在某个bean时，才会实例化这个Bean。
- @ConditionalOnClass，某个class位于类路径上，才会实例化这个Bean。
- @ConditionalOnExpression，当表达式为true的时候，才会实例化这个Bean。
- @ConditionalOnMissingBean，仅在当前上下文中不存在某个bean时，才会实例化这个Bean。
- @ConditionalOnMissingClass，某个class在类路径上不存在的时候，才会实例化这个Bean。
- @ConditionalOnNotWebApplication，不是web应用时才会实例化这个Bean。
- @AutoConfigureAfter，在某个bean完成自动配置后实例化这个bean。
- @AutoConfigureBefore，在某个bean完成自动配置前实例化这个bean。
- @ConfigurationProperties(“spring.redis”) 自动注入属性文件，将配置文件中的属性绑定到实体上。

## 启动流程
![Spring Boot 启动流程图](http://image.wyc1856.club/2019-08-28-13-56-31.png)
> spring boot 服务启动会先初始化一个SpringApplication实例，此阶段只要做以下几件事：
>> 1.把应用启动类添加到PrimarySources集合中   
>> 2.获得应用程序的类型（是否为web项目），后续用来判断是否需要启动内置web容器。  
>> 3.通过SpringFactoriesLoader.loadFactoryNames方法查询classpath:*/META-INF/spring.factories中指定的initializers(初始化器)和listeners(监听器)类名集合，然后通过反射机制进行实例化并设置到springApplication实例内   
>> 4.获得应用的主启动类： new一个RuntimeException并获得其堆栈信息，循环堆栈集合查找方法名为main的类，将其设置为应用主类。

> 创建完实例后调用run方法，此方法主要的作用:
>> 1.初始化SpringApplicationRunListeners实例 (包含一个SpringApplicationLister实例列表和一个Log实例)。   
>> 2.广播服务开始启动事件   
>> 3.通过应用类型获得环境信息，广播环境准备完成事件，配置属性信息，创建应用上下文实例   
>> 4.通过prepareContext方法执行上下文预处理逻辑，包括相关bean的初始化和所有初始化器的执行、广播上下文准备事件、加载上下文信息、广播上下文加载完成事件   
>> 5.执行refreshcontext方法，包含自动装配，注入bean等spring ioc相关操作。自动装配的流程为:通过调用AutoConfigurationImportSelector重写ImportSelector(选择器接口)的selectImports方法，最终还是通过SpringFactoriesLoader.loadFactoryNames方法查询classpath:*/META-INF/spring.factories文件中key为EnableAutoConfiguration的记录，统计出一个自动配置类的类名列表并返回。后续再通过条件注解判断是否需要进行装配，所以引入一些starter即可实现组件的自动配置    
>> 6.广播服务启动完成事件   
>> 7.ApplicationRunner和CommandLineRunner实例执行   
>> 8.广播服务运行中事件，最后返回应用上下文信息

SpringApplicationRunListeners的部分方法会循环调用SpringApplicationRunListener中对应的方法，然后通过实现类EventPublishingRunListener调用SimpleApplicationEventMulticaster进行事件广播，
通过事件类型找到对应的事件监听器实现监听逻辑(还可以启动异步task Executor来实现异步调用);