---
layout:     post
title:      "Hibernate 基础知识"
subtitle:   " \"Hibernate 基础知识\""
date:       2018-02-15 19:25:00
author:     "wmf"
header-img: "img/java.jpg"
catalog: true
tags:
    - java
    - javaee
---
### Hibernate 基础知识
***
#### Hibernate简介
开源持久层对象关系映射框架(Dao)
对JDBC进行了轻量的封装
它将POJO与数据表建立映射关系，是一个全自动的ORM(Object Relational Mapping 关系映射)框架
会自动生成sql语句
当本身功能不够时，可以自行进行扩展
#### 框架一览
Dao层：Hibernate Mybatis
Service层: spring
Web层：springMVC, Struts2
#### 建立一个Hibernate项目
需要引入的jar包
hibernate-release-5.4.1.Final\lib\required下面的所有jar包
数据库驱动jar包mysql-connector-java-5.1.25.jar，commons-beanutils-1.9.3.jar
单元测试jar包junit-4.9.jar
c3p0 jar 包
#### 基本配置
hibernate-release-5.4.1.Final\project\etc下面有一个hibernate.cfg.xml(基本配置),hibernate.properties(参数表)
以下例子链接了一个friends的表
##### hibernate.cfg.xml:
```xml
<!DOCTYPE hibernate-configuration PUBLIC
	"-//Hibernate/Hibernate Configuration DTD 3.0//EN"
	"http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
	<session-factory>
		<property name="hibernate.connection.driver_class">com.mysql.jdbc.Driver</property>
		<property name="hibernate.connection.url">jdbc:mysql://localhost:3306/erm</property>
		<property name="hibernate.connection.username">root</property>
		<property name="hibernate.connection.password">123</property>
		<property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>
		<property name="hibernate.show_sql">true</property>
		<property name="hibernate.format_sql">true</property>
		<property name="hibernate.hbm2ddl.auto">update</property>
		<mapping resource="com/mxq/domain/Friends.hdm.xml"/>
	</session-factory>
</hibernate-configuration>
```
主文件进行配置数据库的地址用户名等,其中```<mapping resource="com/mxq/domain/Friends.hdm.xml"/>```是一个外部定义的表及字段的映射关系,内容如下
##### Friends.hdm.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping PUBLIC 
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
	<class name="com.mxq.domain.Friends" table="friends">
		<!-- 主键建立 -->
		<id name="id" column="id">
			<generator class="native"></generator>
		</id>
                <!-- 其它键 -->
		<property name="name" column="name"></property>
		<property name="url" column="url"></property>
		<property name="qrcode" column="qrcode"></property>
		<property name="qrbrief" column="qrbrief"></property>
	</class>
</hibernate-mapping>
```
对应的dao
##### Friends.java
```java
package com.mxq.domain;

public class Friends {
	private String id;
	private String name;
	private String url;
	private String qrcode;
	private String qrbrief;
	....
}
```
#### 编写代码(不需要写sql只需要操作对象就可以操作数据库)
```java
@Test
public void test1() {
    //加载hibernate核心配置文件
    Configuration configure = new Configuration().configure();
    //创建一个sessionFactory
    SessionFactory buildSessionFactory = configure.buildSessionFactory();
    //通过SessionFactory获取一个Session对象
    Session session = buildSessionFactory.openSession();
    //编写代码
    Friends friends = new Friends();
    friends.setName("2.28james");
    friends.setUrl("www.baidu.com");
    session.save(friends);
    //释放资源
    session.close();
    buildSessionFactory.close();
}
```
#### 自动创建表
hibernate.cfg.xml文件中```<property name="hibernate.hbm2ddl.auto">update</property>```这句话，代表意思没有对应数据表时根据映射关系自动创建表，因此在数据库无表的情况下以上代码依然可以执行成功
这里有几个坑
1. 报错Incorrect column specifier for column 'id'
解决方法，id的映射中加入type="java.lang.String"
```xml
<!-- 主键建立 -->
<id name="id" type="java.lang.String" column="id">
    <generator class="native"></generator>
</id>
```
2. mysql是5.x 会报错 type=MyISAM
修改hibernate.cfg.xml中```<property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>```为```<property name="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</property>```
#### 其它


