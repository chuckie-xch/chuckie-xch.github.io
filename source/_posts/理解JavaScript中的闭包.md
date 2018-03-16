---
title: 理解JavaScript中的闭包
date: 2017-11-27 17:48:54
tags: JavaScript
icon: fa-code
---

# 深入理解JavaScript中的闭包

## 什么是闭包？

**闭包**是词法作用域的体现。简单的说，一个持有外部环境变量的函数就是闭包。闭包的本质是词法作用域和函数当作值传递。

**词法作用域**是只作用域在代码书写或定义的时候就确定的，而动态作用域是在代码执行的时候确定的。词法作用域关注的是**何处声明**，动态作用域关注的是**何处调用**。

看下面代码：
```javascript
function foo() {
   print a;
}
function bar() {
    var a = 3;
    foo();
}
var a = 2;
bar();
```
上述代码词法作用域语言中输出的是2，比如Js中，在动态作用域语言中输出的是3。

我们再来看看闭包的代码：
```javascript
function outer() {
    var local = 2;
    function inner() {
        return local;
    }
    return inner;
}
var fn = outer();
console.log(fn());
```

上面的例子中外部函数outer中定义了内部函数inner，然后将inner函数对象作为值返回。之后执行了outer()函数，在js中outer函数的作用域会被销毁，因为引擎会有垃圾回收器来释放不在使用的内存。
因为这里将函数作为返回值了，产生了闭包，因为闭包的存在，我们依然能够访问到local变量，所有outer的词法作用域依然存在。因此也就证明了，**闭包可访问所在的词法作用域并且拥有更长的生命周期，保持对当前词法作用域的引用。**


