---
layout: idea
title: maven导入本地jar导致NoClassDefFoundError
date: 2017-11-30 19:42:20
tags: Java异常
---

# maven导入本地jar导致NoClassDefFoundError

**方法一**
将jar包打包到本地仓库
```java
mvn install:install-file -Dfile=D:/internetenv/myLib/alipay/alipay-trade-sdk.jar -DgroupId=com.alibaba -DartifactId=aliPayDemo -Dversion=2.0 -Dpackaging=jar

mvn install:install-file -Dfile=D:/internetenv/myLib/alipay/alipay-sdk-java20160302220055.jar -DgroupId=com.alibaba -DartifactId=aliPay -Dversion=2.0 -Dpackaging=jar

mvn install:install-file -Dfile=D:/internetenv/system/lib/wxpay-scanpay-java-sdk-1.0.jar -DgroupId=com.tencent -DartifactId=wxpay-scanpay-java-sdk -Dversion=1.0 -Dpackaging=jar
```

**方法二**
配置Maven插件

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <executions>
        <execution>
            <id>copy-dependencies</id>
            <phase>compile</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>
                <outputDirectory>${project.build.directory}/${project.build.finalName}/WEB-INF/lib</outputDirectory>
                <includeScope>system</includeScope>
            </configuration>
        </execution>
    </executions>
</plugin>
```