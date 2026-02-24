---
title: Spring boot 自动装配实现的原理 – 文字简述版
author: Bridge Li
type: post
date: 2021-08-29T03:34:29+00:00

categories:
  - Java
tags:
  - 自动装配
---
1. 当启动 Spring boot 应用程序的时候，会先创建 SpringApplication 的对象，在对象的构造方法中会进行某些参数的初始化工作，最主要的是判断当前应用程序的类型以及初始化器和监听器，在这个过程中会加载整个应用程序的 spring.factories 文件，将文件的内容放到缓存对象中，方便后续获取。

2. SpringApplication 对象创建完成之后，开始执行 run 方法，来完成整个启动，启动过程中最主要的有两个方法，第一个叫做 prepareContext，第二个叫做 refreshContext，在这两个关键步骤中完成了自动装配的核心功能，前面的处理逻辑包含了上下文对象的创建，banner 的打印，异常报告器的准备等各个准备工作，方便后续来进行调用。

3. 在 prepareContext 方法中主要完成的是对上下文对象的初始化操作，包含了属性值的设置，比如环境对象，在整个过程中有一个非常重要的方法，叫做 load，load 主要完成一件事，将当前启动类作为一个 beanDefinition 注册到 registry 中，方便后续在进行 BeanFactoryPostProcessor 调用执行的时候，找到对应的主类，来完成 @SpringBootApplication、@EnableAutoConfiguration 等注解的解析工作。

4. 在 refreshContext 方法中会进行整个容器的刷新过程，会调用 Spring 中的 refresh 方法，refresh 中有 13 个非常关键的方法，来完成整个 Spring 应用程序的启动，在自动装配过程中，会调用 invokeBeanFactoryPostProcessor 方法，在此方法中主要对 ConfigurationClassPostProcessor 类的处理，他是 BeanFactoryPostProcessor 的子类也是，BeanDefinitionRegistryPostProcessor 的子类，在调用的时候会先调用 BeanDefinitionRegistryPostProcessor 中的 postProcessBeanDefinitionRegistry 方法，然后调用 BeanFactoryPostProcessor 中的 postProcessBeanFactory 方法，在执行 postProcessBeanDefinitionRegistry 方法的时候会解析处理各种注解，包含 @PropertySource、@ComponentScan、@ComponentScans、@Bean、@Import 等注解，最主要的是 @Import 注解的解析。

5. 在解析 @Import 注解的时候，会有一个 getImport 的方法，从主类开始递归解析注解，把所有包含 @Import 的注解都解析道，然后在 processImport 方法中对 Import 的类进行分类，此处最主要的是识别 AutoConfigurationImportSelect 归属于 ImportSelect 的子类，在后续过程中会调用 deferredImportSelectorHandler 中的 process 方法，来完善 EnableAutoConfiguration 的加载。