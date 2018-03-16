---
layout: intellij
title: idea安装配置
date: 2017-12-07 13:40:22
tags: 开发工具
icon: fa-sun-o
---

# intellij idea 2017.3安装配置

## 安装

从官网下载旗舰版安装文件,一路next，安装成功。

## 破解

 license server下输入
```url
http://idea.iteblog.com/key.php
```

## 配置

点击Configuration--->defaultSettings。
**配置maven**
![maven配置](http://upload.ouliu.net/i/20171207135218vb9aa.png)

## tomcat内存不足配置

在Run/Debug configuration 的你要运行行的tomcat里面的 
vm options里面输入
```code
-server -XX:PermSize=128M -XX:MaxPermSize=256m
或
-server -XX:PermSize=521M -XX:MaxPermSize=1024m
```

