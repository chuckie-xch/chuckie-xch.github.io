---
title: Java代码规范
date: 2018-03-7 19:02:00
tags: 代码规范
---

代码拥有统一的格式和规范，既便于代码的逻辑清晰，又便于维护，好的编码规范可以尽可能的减少一个软件的维护成本。下面从三个方面进行约束：服务端口规范，方法命名规范，接口定义规范。

<!-- more -->

### 服务端口规范

* 每个业务模块定义一个模块ID，如账户模块：20
* Web模块：
  * Server Port：模块ID+80，如2080
  * Manager Port：模块ID+81，如2181
* Server模块：
  * Server Port：模块ID+09，如2009
  * Dubbo Port：模块ID+08，如2008
* MySQL：模块ID+06，如2006



### 方法命名规范

总体上根据阿里巴巴编码规范进行约束。

* 新增：save...
* 修改：update...
* 查询：find...，如findByName
* 查询所有：findAll
* 分页查询：find...ByPage
* 删除：delete
* 批量删除：batchDelete
* 业务方法：动词+名词组合
* kafka主题：topic-业务名称
* kafka消费组：group-模块名称



### 接口定义规范

统一返回格式：

* BaseResult
* GeneralReulst<T>
* ListResult

其他：

* 新增接口理应返回新增对象。
* 不允许把Json，Map这类对象传到Service中去，也不允许Service返回Json，Map这类对象。
* Controller尽量少打日志，Controller里的日志通过AOP去打，除非第三方接口回调。

