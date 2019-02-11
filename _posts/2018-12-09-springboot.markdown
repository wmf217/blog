---
layout:     post
title:      "springboot 学习笔记"
subtitle:   " \"springboot 学习笔记\""
date:       2018-12-09 19:25:00
author:     "wmf"
header-img: "img/post-bg-js-version.jpg"
catalog: true
tags:
    - java
    - spring
---
### springboot 学习笔记
##### 微服务
一个项目 可以由多个 小型服务构成
##### sprint boot 可以简化开发 微服务模块
1. 简化j2ee开发
2. 整个spring技术栈的整合(springmvc spring)
3. 整个j2ee技术的整合 (mybatis redis)
##### 准备
jdk: 
1. JAVA_HOME: jdk的根目录
2. path: %JAVA_HOME%/bin
3. classpath: .;%JAVA_HOME%/lib
maven: 
1. MAVEN_HOME: maven的根目录
2. path: %MAVEN_HOME%/bin
3. 配置maven本地仓库， mvn 根目录 /conf/setting.xml localRepository
###### springboot 开发工具
1. Eclipse(STS插件) > STS
2. Intellij IDEA
###### 目录结构resources
1. static静态文件
2. templates: 模板文件(引擎 freemarker, thymeleaf,默认不支持jsp)
3. application.properties springboot配置文件
###### springboot web
1. 原web项目webContext web.xml > war > tomcat
2. springboot 内置了tomcat, 并且不需要打成war包， 可以在application.properties对端号等服务信息进行配置
###### springboot 场景
将各各应用/三方框架设置成了一个个场景 starter, 以后要用哪个选哪个场景，选完之后springboot将所有依赖自动注入，例如 选择“web”, 会将全部依赖自动注入（tomcat json）
###### springboot 注解
@SpringBootApplication: springboot 的主配置类，该注解包含
1. @SpringBootConfiguration: 包含@Configuration， 代表配置类， 用配置类代替配置文件，
* 该注解表示该类是一个配置类
* 自动纳入spring容器里
```java
@Configuration//表示A是一个配置类
public class A
```
2. @EnableAutoConfiguration：该注解使springboot可以自动配置(约定由于配置):可以找到@SpringBootConfiguration所在的类，再spring boot 启动时，会根据META-INF/spring.factories找到相应的第三方依赖，并将这些依赖引入本项目

