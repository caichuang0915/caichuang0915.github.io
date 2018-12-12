---
title: Maven基础 - 打包调用异常
tags:
  - Maven
---

Maven基础学习配置

## Maven 打包遇见的问题

一个项目依赖以另一个项目时引入其依赖,想要调用其依赖的jar包中的类，则打包方式需使用maven的打包方式，不能使用spring-boot的打包方式 可以使用@Component将jar包中的类扫描注入到当前项目的IOC容器中

<!-- more -->

```java
<build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.1</version>
        <configuration>
          <source>1.8</source>
          <target>1.8</target>
        </configuration>
      </plugin>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
```

而spring项目使用jar包启动时则需要使用spring-boot的打包方式
```java
<build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
```

两种方式打包的目录结构：

- maven打包

![maven打包的目录结构](http://tupelo.top/maven.png)

- spring-boot打包

![maven打包的目录结构](http://tupelo.top/spring.jpg)





