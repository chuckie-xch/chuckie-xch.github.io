---
title: Angualr最佳实践
date: 2018-03-16 10:16:43
tags: Angualr
---

根据自身的项目经验，总结Angular的一些最佳实践，旨在提高开发效率，提高代码质量。
<!-- more -->

## 单一职责

- 一个文件定义一样东西，比如一个组件、一个服务、一个管道、一个指令
- 每个文件最多不要超过400行
- 定义功能单一的函数
- 一个函数最多不要超过75行

## 命名规范

- 文件名采用feature.type.**，feature表示特性，type表示类型


- - 模块用 .module.ts
  - 路由模块用 -routing.module.ts
  - 组件用 .component.ts|html|css
  - 服务用 .service.ts
  - 管道用 .pipe.ts
  - 指令用 .directive.ts
  - 类型用 .model.ts
  - 数据用 .data.ts


- 用"-"来分割单词，比如hero-list.component.ts
- 单元测试文件名保持和测试对象一致，并以.spec.ts结尾
- 端到端测试文件名保持和测试对象一致，并以.e2e-spec.ts结尾
- 类名用大写驼峰规则，并且保持跟文件名的一致


- - 模块：比如app.module.ts定义的类名为AppModule
  - 路由模块：比如app-routing.module.ts定义的类名为AppRoutingModule
  - 组件：比如hero-list.component.ts定义的类名为HeroListComponent
  - 服务：比如logger.service.ts定义的类名为LoggerService
  - 管道：比如address.pipe.ts定义的类名为AddressPipe
  - 指令：比如highlight.directive.ts定义的类名为HighlightDirective
  - 类型：按模块来划分，一个.model.ts定义多个类型
  - 数据：比如address-book.data.ts定义的变量名为addressBook


- 指令选择器的命名采用小写驼峰规则，比如clickOutSide
- 组件选择器的命名采用分隔符“-”连接小写字母的形式，比如hero-list

## 代码规范

- 类命名采用大写驼峰规则
- 常量定义用const，并且全部大写，如果有多个单词，用“_”连接，比如HERO_URL
- 支持ES6的环境下，禁止使用var定义变量
- 变量命名尽量控制在3个单词以内，有常见缩写形式的单词可采用缩写形式
- 属性和方法名用小写驼峰，私有属性和方法不建议以“_”为前缀
- 建议用空一行的方式来区分第三方库的导入和项目本身文件的导入

## 模块

- 为每个特性创建一个模块，并且保持文件夹命名和模块命名一致
- 共享模块建议用SharedModule命名，放在app/shared/shared.module.ts中
- 在共享模块中定义复用的组件、指令和管道，避免在共享模块中定义服务
- 在共享模块中导入所有必需的模块，比如CommonModule和FormsModule，导出所有复用的模块、组件、指令、管道
- 考虑将只用一次的类放在核心特性模块中，并且仅在根模块中导入，建议写guard.ts来保证
- 将单例服务放在核心特性模块中，比如ExceptionService和LoggerService
- 在核心特性模块中导入所有必需的模块，比如CommonModule和FormsModule，导出所有定义的组件，服务等
- 将全局仅用一次的组件放在核心特性模块，然后只在AppModule中导入，比如HeaderComponent和FooterComponent
- 独立的特性模块可以做成懒加载模块，避免在任何地方导入懒加载模块，不然模块就会直接加载

## 组件

- 当模板／样式超过3行时，写成单独文件
- 删除无样式定义的样式文件
- 模板／样式的定义与组件命名保持一致
- 导入时使用相对路径
- 用装饰器@Input和@Output来修改输入输出数据，而不是使用元数据中的inputs和outputs属性
- 按照变量，构造器，生命周期函数，一般方法的顺序定义；一般方法按页面的功能模块放一起，被调用的方法写在后面；变量和方法均先公有后私有排列；
- 将可复用的业务逻辑放在服务中
- 将展示逻辑放在组件的类中，而不是在模板中

## 服务

- 跟函数一样，一个服务只有一个目的
- 把服务注入在最高层的组件／模块中，使得该单例服务能在子组件、子模块中共享
- 使用@Injectable装饰服务类，而不是@Inject装饰参数

## 生命周期

- 需要生命周期钩子时，实现相关的接口，可有效防止错误

辅助工具

- 使用IDE的代码片段工具来快速生成具有一致性的代码片段，比如给VS Code安装[snippets](http://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Djohnpapa.Angular2)

利用好TypeScript类型

- 限制类型——通过枚举来代替

```typescript
interface Order {  
    status: 'pending' | 'approved' | 'rejected';
}

enum Status {
  Pending = 1,
  Approved = 2,
  Rejected = 3
}

interface Order {
  status: Statuses;
}
```

* 组件生命周期钩子：顺序-

1. ngOnChanges() ：当Angular（重新）设置数据绑定输入属性时响应。 

该方法接受当前和上一属性值的SimpleChanges对象当被绑定的输入属性的值发生变化时调用，首次调用一定会发生在ngOnInit()之前。

2. ngOnInit():在Angular第一次显示数据绑定和设置指令/组件的输入属性之后，初始化指令/组件。只调用一次
3. ngDoCheck():检测，并在发生Angular无法或不愿意自己检测的变化时作出反应。在每个Angular变更检测周期中调用，ngOnChanges()和ngOnInit()之后。
4. ngAfterContentInit()：当把内容投影进组件之后调用。第一次ngDoCheck()之后调用，只调用一次。只适用于组件。
5. ngAfterContentChecked()：每次完成被投影组件内容的变更检测之后调用。ngAfterContentInit()和每次ngDoCheck()之后调用只适合组件。
6. ngAfterViewInit()：初始化完组件视图及其子视图之后调用。第一次ngAfterContentChecked()之后调用，只调用一次。只适合组件。
7. ngAfterViewChecked()：每次做完组件视图和子视图的变更检测之后调用。ngAfterViewInit()和每次ngAfterContentChecked()之后调用。只适合组件。
8. ngOnDestroy：当Angular每次销毁指令/组件之前调用并清扫。 在这儿反订阅可观察对象和分离事件处理器，以防内存泄漏。在Angular销毁指令/组件之前调用。