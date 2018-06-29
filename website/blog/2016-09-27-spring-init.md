---
layout: post
title:  "spring mvc  DispatcherServlet ContextLoaderListener"
date:  2016-09-27 15:50:00 +0800
categories: spring
tags: spring
keywords: spring,web1992,spring mvc
---

> `spring mvc` `DispatcherServlet` `ContextLoaderListener` 以及`Filter` 总结
> 最近在使用`spring mvc` 进行web项目的开发
> 以前只是会用，这次对使用中`疑惑的地方`进行下总结

----------

<!--more-->

- 1.[DispatcherServlet 的作用](#v1)
- 2.[DispatcherServlet 的初始化](#v2)
- 3.[DispatcherServlet 的配置](#v3)
- 4.[ContextLoaderListener 配置](#v4)
- 5.[ContextLoaderListener 的作用](#v5)
- 6.[ContextLoaderListener类的继承关系](#v6)
- 7.[参考的文章](#v7)


1.`DispatcherServlet`的作用：<a name="v1"></a>
----

在 `DispatcherServlet javadoc` 中有这句话

```java
	Central dispatcher for HTTP request handlers/controllers, e.g. for web UI controllers or HTTP-based remote service
	exporters. Dispatches to registered handlers for processing a web request, providing convenient mapping and exception
	handling facilities.

	Central dispatcher // DispatcherServlet是一个调度中心，主要处理http请求，url映射，异常处理等
```	


2.`DispatcherServlet` 的初始化： <a name="v2"></a>
---

> DispatcherServlet 是一个`servlet` 继承了我们熟悉的 `HttpServlet`
> 它的初始化时在web容器创建时进行的，如果配置了 Filter，可以看到如下日志

```java
	[INFO] Initializing log4j from [D:\github\javas\java_note\app_assembly\target\assembly_app_war\webapp\WEB-INF\log4j.xml]
	[INFO] Initializing Spring root WebApplicationContext
	Root WebApplicationContext: initialization started // WebApplicationContext初始化开始
	// ...
	Root WebApplicationContext: initialization completed in 844 ms // WebApplicationContext初始化结束
	// ... 
	DemoFilter init // 初始化基于servlet规范的过滤器
	DemoFilter2 init // 初始化基于servlet规范的过滤器
    [INFO] Initializing Spring FrameworkServlet 'dispatcherServlet' // 初始化dispatcherServlet
	// ...
	[INFO] Started Jetty Server
	
	1,先初始化 log4j
	2,再初始化 WebApplicationContext
	3,初始化 Filter 基于Servlet 规范的过滤器
	4,初始化 dispatcherServlet 
	5,web 容器启动完成
```
	

3.`DispatcherServlet` 的配置： <a name="v3"></a>
---

```xml
		<servlet>
	        <servlet-name>dispatcherServlet</servlet-name>
	        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	        <init-param>
	            <param-name>contextConfigLocation</param-name>
	            <param-value>classpath:META-INF/spring/servlet-context.xml</param-value>
	        </init-param>
	        <load-on-startup>1</load-on-startup>
	    </servlet>
	    <servlet-mapping>
	        <servlet-name>dispatcherServlet</servlet-name>
	        <url-pattern>/</url-pattern>
	    </servlet-mapping>

```		

> a,在web.xml 使用 `servlet`标签 进行配置

> b, `DispatcherServlet` 告诉spring 配置文件的位置
> servlet-context.xml 配置文件配置spring mcv,拦截器`HandlerInterceptor`等 

> DispatcherServlet 类的继承关系:
>
> 实线的箭头是继承`extends`，虚线的箭头是类的实现`implements`

![](https://i.imgur.com/h3o9bYP.jpg)



4.`ContextLoaderListener`配置： <a name="v4"></a>
---

```xml
	<!-- spring 配置文件 -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>
			classpath:META-INF/spring/servlet-context.xml,
			classpath:META-INF/spring/spring-context.xml,
		</param-value>
	</context-param>
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
```


5.`ContextLoaderListener` 的作用 <a name="v5"></a>
---
`ContextLoaderListener` 继承了 `ServletContextListener`代码如下：

	public class ContextLoaderListener extends ContextLoader implements ServletContextListener {
	// ...	
	}


`ContextLoaderListener`只是一个对外暴露的实现类，具体的实现是在 `ContextLoader` 中
	
`ContextLoader` 使用 默认的 `XmlWebApplicationContext` 加载Bean,`ContextLoader.properties` 配置了默认的 bean 加载类,如下：

	org.springframework.web.context.WebApplicationContext=org.springframework.web.context.support.XmlWebApplicationContext


6.`ContextLoaderListener` 类的继承关系 <a name="v6"></a>
---

![](https://i.imgur.com/0mWshgN.jpg)


---------
7.参考的文章： <a name="v7"></a>

- [参考1](http://blog.csdn.net/agileclipse/article/details/9014683)
- [参考2](http://www.cnblogs.com/JesseV/archive/2009/11/17/1605015.html)
- [参考3](http://www.cnblogs.com/hellojava/archive/2012/12/26/2833840.html)
- [参考4-filter](http://tianweili.github.io/blog/2015/01/26/java-filter/)


[#v1]:v1
[#v2]:v2
[#v3]:v3
[#v4]:v4
[#v5]:v5
[#v6]:v6
[#v7]:v7



