---
title: SpringMVC源码分析
date: 2018-03-15 18:34:52
tags:
---

# SpringMVC源码分析



## Quick Start

### pom中引入依赖

<!-- more -->

```xml
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-webmvc</artifactId>
  <version>4.3.11.RELEASE</version>
</dependency>

<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>javax.servlet-api</artifactId>
  <version>3.1.0</version>
  <scope>provided</scope>
</dependency>
```



### 配置Web.xml和application.xml

`web.xml`:

```xml
<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>classpath:spring-mvc.xml</param-value>
</context-param>

<servlet>
  <servlet-name>spring</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:spring-mvc.xml</param-value>
  </init-param>
  <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
  <servlet-name>spring</servlet-name>
  <url-pattern>/</url-pattern>
</servlet-mapping>

<listener>
  <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

`spring-mvc.xml:`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/context
http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.bestcode.springmvc.controller"/>

    <context:annotation-config></context:annotation-config>
</beans>
```



### Controller代码

```java
@Controller
@RequestMapping
public class IndexController {

    @RequestMapping("/index")
    @ResponseBody
    public String index() {
        return "hello , Spring Mvc";
    }
}
```



## SpringMVC核心——DispatcherServlet

### DispatcherServlet架构图

[![DispatcherServlet.png](https://i.loli.net/2018/03/15/5aaa1720f094f.png)](https://i.loli.net/2018/03/15/5aaa1720f094f.png)

可以看到三个核心类`DispatcherServlet`，`FramewordServlet`，`HttpServletBean`，我们先来看`HttpServlet`的直接子类`HttpServletBean`都做了哪些事：

`HttpServletBean`类的定义：

```java
public abstract class HttpServletBean extends HttpServlet implements EnvironmentCapable, EnvironmentAware 
```

[![HttpServletBean.png](https://i.loli.net/2018/03/15/5aaa1a04efc12.png)](https://i.loli.net/2018/03/15/5aaa1a04efc12.png)

`HttpServletBean`中的核心`init`方法:

```java
@Override
public final void init() throws ServletException {
   if (logger.isDebugEnabled()) {
      logger.debug("Initializing servlet '" + getServletName() + "'");
   }

   // 从init-param中解析contextConfigLocation，加载spring配置文件
   PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
   if (!pvs.isEmpty()) {
      try {
         // 包裹DispatcherServlet对象的BeanWrapperImpl
         BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
         // 获取ServletContextResourceLoader
         ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
         bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
         initBeanWrapper(bw);
         bw.setPropertyValues(pvs, true);
      }
      catch (BeansException ex) {
         if (logger.isErrorEnabled()) {
            logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex);
         }
         throw ex;
      }
   }

   // Let subclasses do whatever initialization they like.
   initServletBean();

   if (logger.isDebugEnabled()) {
      logger.debug("Servlet '" + getServletName() + "' configured successfully");
   }
}
```

可以看出，容器启动的时候会加载spring配置元数据，注册用户自定义的property editor。

然后，初始化ServletBean，由子类负责实现，我们来看下FramewordServlet中的实现，在此之前我们先来看看FrameworkServlet中的一个非常重要的属性：

```java
private WebApplicationContext webApplicationContext;
```
下面是`WebApplicationContext`的类图：

![L2mOO.png](https://s1.ax2x.com/2018/03/15/L2mOO.png)

这里终于看到了我们熟悉的IOC容器接口`ApplicationContext`，说明，容器的加载获取，这在这个方法中加载的。

那是不是就是这个`initServletBean`方法所做的事情呢，下面我们验证我们的猜想，看看`FrameworkServlet`中`initServletBean`到底怎么实现的：


```java
protected final void initServletBean() throws ServletException {
   getServletContext().log("Initializing Spring FrameworkServlet '" + getServletName() + "'");
   if (this.logger.isInfoEnabled()) {
      this.logger.info("FrameworkServlet '" + getServletName() + "': initialization started");
   }
   long startTime = System.currentTimeMillis();

   try {
      // 初始化WebApplicationContext 拿到WebApplicationContext的实现类XmlWebApplicationContext
      this.webApplicationContext = initWebApplicationContext();
      initFrameworkServlet();
   }
   catch (ServletException ex) {
      this.logger.error("Context initialization failed", ex);
      throw ex;
   }
   catch (RuntimeException ex) {
      this.logger.error("Context initialization failed", ex);
      throw ex;
   }

   if (this.logger.isInfoEnabled()) {
      long elapsedTime = System.currentTimeMillis() - startTime;
      this.logger.info("FrameworkServlet '" + getServletName() + "': initialization completed in " +
            elapsedTime + " ms");
   }
}

```
可以看到通过`initWebApplicationContext`方法获取到`WebApplicationConext`（`XmlWebApplicationContext`）

```java
protected WebApplicationContext initWebApplicationContext() {
   WebApplicationContext rootContext =
         WebApplicationContextUtils.getWebApplicationContext(getServletContext());
   WebApplicationContext wac = null;

   if (this.webApplicationContext != null) {
      // A context instance was injected at construction time -> use it
      wac = this.webApplicationContext;
      if (wac instanceof ConfigurableWebApplicationContext) {
         ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
         if (!cwac.isActive()) {
            // The context has not yet been refreshed -> provide services such as
            // setting the parent context, setting the application context id, etc
            if (cwac.getParent() == null) {
               // The context instance was injected without an explicit parent -> set
               // the root application context (if any; may be null) as the parent
               cwac.setParent(rootContext);
            }
            configureAndRefreshWebApplicationContext(cwac);
         }
      }
   }
   if (wac == null) {
      // No context instance was injected at construction time -> see if one
      // has been registered in the servlet context. If one exists, it is assumed
      // that the parent context (if any) has already been set and that the
      // user has performed any initialization such as setting the context id
      wac = findWebApplicationContext();
   }
   if (wac == null) {
      // No context instance is defined for this servlet -> create a local one
      wac = createWebApplicationContext(rootContext);
   }

   if (!this.refreshEventReceived) {
      // Either the context is not a ConfigurableApplicationContext with refresh
      // support or the context injected at construction time had already been
      // refreshed -> trigger initial onRefresh manually here.
      onRefresh(wac);
   }

   if (this.publishContext) {
      // Publish the context as a servlet context attribute.
      String attrName = getServletContextAttributeName();
      getServletContext().setAttribute(attrName, wac);
      if (this.logger.isDebugEnabled()) {
         this.logger.debug("Published WebApplicationContext of servlet '" + getServletName() +
               "' as ServletContext attribute with name [" + attrName + "]");
      }
   }

   return wac;
}

```

当`wac==null`时，会调用`createWebApplicationContext(rootContext)`

```java
protected WebApplicationContext createWebApplicationContext(ApplicationContext parent) {
   Class<?> contextClass = getContextClass();
   if (this.logger.isDebugEnabled()) {
      this.logger.debug("Servlet with name '" + getServletName() +
            "' will try to create custom WebApplicationContext context of class '" +
            contextClass.getName() + "'" + ", using parent context [" + parent + "]");
   }
   if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
      throw new ApplicationContextException(
            "Fatal initialization error in servlet with name '" + getServletName() +
            "': custom WebApplicationContext class [" + contextClass.getName() +
            "] is not of type ConfigurableWebApplicationContext");
   }
   ConfigurableWebApplicationContext wac =
         (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);
   // 设置环境，这里是StandardServletEnvironment
   wac.setEnvironment(getEnvironment());
   // 设置父Context
   wac.setParent(parent);
   // 设置配置位置：classpath:spring-mvc.xml
   wac.setConfigLocation(getContextConfigLocation());

   configureAndRefreshWebApplicationContext(wac);

   return wac;
}
```









