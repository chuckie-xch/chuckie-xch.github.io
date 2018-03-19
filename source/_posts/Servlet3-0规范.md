---
title: Servlet3.0规范
date: 2018-01-29 18:52:44
desc: Servlet
---


**1.什么是Servlet?**

`Servlet`是一个基于Java技术的Web组件，生成动态的内容，并通过容器管理。

**2.什么是Servlet容器？**

**Servlet容器**是Web服务或应用服务的一部分，它提供了发送请求，响应，解码基于MIME的请求，格式化基于mime的响应。并且管理Servlet的声明周期。

<!-- more -->

### Servlet API

**Servlet**接口是Java Servlet API的核心抽象。所有Servlet接口必须直接实现或者继承实现了该接口的类。Java Servlet API提供了两个实现该接口的类。**GenericServlet**和**HttpServlet**。大多数场合，开发者都会继承**HttpServlet**来实现Servlet接口。



### 处理请求的方法

基础的Servlet提供了一个**service**方法来处理客户端请求。每个请求都会被Servlet容器**路由**到对应的Servlet实例上调用其service方法。





