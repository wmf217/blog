---
layout:     post
title:      "springboot 学习笔记"
subtitle:   " \"springboot 学习笔记\""
date:       2018-12-09 19:25:00
author:     "wmf"
header-img: "img/in-post/laravel.jpg"
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
2. @EnableAutoConfiguration：该注解使springboot可以自动配置(约定由于配置):
* ```@AutoConfigurationPackage```可以找到@SpringBootConfiguration所在包名，自动扫描包和子包，全部纳入spring容器，不用去配置scan，不用写类似```<context:component-scan base-package="com.company.demo1"/>```的配置
具体实现代码：
    ```java
    (new AutoConfigurationPackages.PackageImport(metadata)).getPackageName()
    ```
    其中metadata就是有@SpringBootApplication注解的类的reference，获取到的包名(getPackageName)就是有该注解的类所在的包名
* ```@Import({AutoConfigurationImportSelector.class})```，再spring boot 启动时，会根据META-INF/spring.factories找到相应的第三方依赖，并将这些依赖引入本项目
具体实现代码：
    ```java
        List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
    ```
    ```java
        Enumeration<URL> urls = classLoader != null ? classLoader.getResources("META-INF/spring.factories") : ClassLoader.getSystemResources("META-INF/spring.factories");
    ```
总结：编写项目时，一般会对自己写的代码以及三方依赖进行配置，但是springboot可以自动配置，其中自己写的代码通过```@AutoConfigurationPackage```自动配置，三方依赖依靠AutoConfigurationImportSelector.class进行配置
###### application.properties配置文件
可以对约定好的一些配置进行自定义修改，例如
```properties
server.port=8888
```
修改端口号
###### yml
yml不同于xml不是一个标记文档
注意：
* : 后面一定要接空格
* 缩进代表层级
* yml字符串默认可以不加引号，如果加上""有转义的效果，其它没有
* yml中的[]可以省略，{}不能省
写法：
```yml
server:
  port: 9999
 ```
 ###### yml注入
 entity:
 ```java
@Component
@ConfigurationProperties(prefix = "student") //注入方式
public class Student {
    private String name;
    private int age;
    private Boolean sex;
    private Date birthday;
    private Map<String, Object> location;
    private String[] hobbies;
    private String[] skills;
    private Pet pet;
 ```
 ```java
public class Pet {
    private String nickname;
    private String strain;
 ```
application.yml:
 ```yml
student:
  name: zx
  age: 23
  sex: true
  birthday: 2019/03/24
  location: {province: 山西, city: 大同, zone: 金山区}
  hobbies: [football, 篮球]
  skills:
    - 编程
    - 英语
  pet:
    nickname: 旺财
    strain: 金毛
 ```
 Test:
 ```java
@Autowired
Student student;
@Test
public void contextLoads() {
    System.out.println(student.toString());
}
 ```
 输出结果
 ```Student{name='zx', age=23, sex=true, birthday=Sun Mar 24 00:00:00 CST 2019, location={province=山西, city=大同, zone=金山区}, hobbies=[football, 篮球], skills=[编程, 英语], pet=Pet{nickname='旺财', strain='金毛'}}```
 ###### application.properties与yml互补
 application.yml:
 ```yml
 student:
 # name: zx
 # age: 23
 ```
 application.properties:
```properties
#server.port=8888
student.name=wmf
student.age=18
```
这样可以互补，application.properties同样可以配置，而且两个可以互补
###### ConfigurationProperties和Value
以上的Student类是通过ConfigurationProperties注解绑定到student(yml或properties中定义)，同时也可以用String @Value注解注入，<font color="red">但优先级是ConfigurationProperties高</font>，同样可以达到互补的目的
注意：
* ConfigurationProperties可以使用松散语法例如nickName可以改成nick-name
* @Value可以SpEl表达式
    ```java
    public class Student {
        @Value("${student.username}")
        private String name;
    }
    ```
* JSR303校验ConfigurationProperties是支持的(会报错)，value不支持(不会报错)
    ```java
    @ConfigurationProperties(prefix = "student")
    @Validated
    public class Student {
        @Value("${student.username}")
        private String name;
        @Email
        private String email;
    ```
    ```yml
    student:
    username: 傻七
    email: 123131是是是
    ```
    此时编译会报错，不是邮箱格式
* ConfigurationProperties可以注入简单类型
###### @PropertySource注解
以上application.properties和application.yml都是boot自动加载的，如果想加载其它的文件可以用@PropertySource注解注解，但该注解暂时只支持properties文件
###### @ImportResource
* spring自动装配/自动配置
* spring等配置文件，默认会被spring boot自动配置好
* 如果自己想写spring配置文件，又想让springboot识别比如写一个spring.xml
    ```xml
    <bean id="studentService" class="com.wmf.hello.service.StudentService">
    </bean>
    ```
    业务逻辑层
    ```java
    public class StudentService {
    }
    ```
    Test
    ```java
    @Autowired
    ApplicationContext context; //string 容器
    @Test
    public void test () {
        StudentService studentService = (StudentService) context.getBean("studentService");
        System.out.println(studentService);
    }
    ```
    输出结果```No bean named 'studentService' available```
* 解决方法
    ```java
    @ImportResource(locations = {"classpath:spring.xml"})
    @SpringBootApplication
    public class HelloApplication {
        public static void main(String[] args) {
            SpringApplication.run(HelloApplication.class, args);
        }
    }
    ```
    加上```@ImportResource(locations = {"classpath:spring.xml"})```程序就可以识别配置文件了，但是这种做法不推荐，使用注解配置最好
##### 使用配置类
* 上面的做法不推荐
* 推荐使用@Configuration和@Bean
    com.wmf.hello.Config.AppConfig:
    ```java
    @Configuration
    public class AppConfig {
        @Bean
        public StudentService studentService () {
            StudentService studentService = new StudentService();
            return studentService;
        }
    }
    ```
    这种写法与上面的spring.xml效果一样
##### 全局配置文件中的占位符
* application.yml
    ```yml
    student:
        username: ${random.value}
    ```
    结果生成一串随机数
* application.yml和application.properties互通
    application.properties:
    ```properties
    student.user.name=wmf
    ```
    application.yml:
    ```yml
    student:
        username: ${student.user.name}
    ```
##### 多环境设置和切换(profile)
默认情况下boot读取的是application.properties配置文件
* properties多个文件：
    application-dev.properties 开发
    application-test.properties 测试
    application.properties 正式
    比如现在要切换开发环境：
    直接在application.properties配置：
    ```properties
    spring.profiles.active=dev
    ```
    那么boot此时遵循的就是application-dev.properties配置的内容
* yml多环境
    profiles是关键
    ```yml
    server:
        port: 8888
    spring:
        profiles:
            active: dev #选用开发配置
    ---
    server:
        port: 7777
    spring:
        rofiles: dev
    ---
    server:
        port: 4444
    spring:
        profiles: test
    ```
###### 命令行动态切换环境
* 编辑器命令行方式
    run with configuration
    idea: 直接在active profiles里加dev
    eclipse: run with configuration->arguments填写```--spring.profiles.active=dev```
* jar包方式
##### 配置项目名
```server.servlet.context-path=/wmf```
##### 外部配置文件
比如说放置D盘一个配置文件application.properties内容
```properties
server.port=8888
server.servlet.context-path=/web
student.username=wmf
```
idea->run->edit Configurations->override parameters
加上spring.config.location：D:/application.properties
再次运行，此时执行结果
```
Tomcat initialized with port(s): 8888 (http)
Tomcat started on port(s): 8888 (http) with context path '/web'
```
##### IDEA 打JAR包
https://blog.csdn.net/sinat_33201781/article/details/80264828
其中maven project找不到的话View > Tool Windows > Maven
执行jar包 java -jar jar包名
##### 通过外部配置文件执行jar包
jar包 java -jar jar包名 --spring.config.location=D:/application.properties
##### 项目运行参数（补救）
idea->run->edit Configurations->override parameters
加上server.port：9999
此时再运行会执行9999端口号而覆盖配置文件
##### springboot 日志
常用的日志框架：UCL JUL jboss-logging logback log4j log4j2 slf4j
springboot默认选用slf4j logback
springboot默认配置好了日志
```java
Logger logger = LoggerFactory.getLogger(HelloApplicationTests.class);
@Test
public void testLog () { //日志级别
    logger.trace("trace...");
    logger.debug("debug...");
    logger.info("info...");
    logger.warn("warn....");
    logger.error("error...");
}
```
输出
```
2019-03-27 19:58:09.854  INFO 7480 --- [           main] com.wmf.hello.HelloApplicationTests      : info...
2019-03-27 19:58:09.854  WARN 7480 --- [           main] com.wmf.hello.HelloApplicationTests      : warn....
2019-03-27 19:58:09.854 ERROR 7480 --- [           main] com.wmf.hello.HelloApplicationTests      : error...
```
日志级别：
```java
public enum LogLevel {
    TRACE,
    DEBUG,
    INFO,
    WARN,
    ERROR,
    FATAL,
    OFF;

    private LogLevel() {
    }
}
```
默认info及以上会打印
如果要修改application.properties设置```logging.level.主配置类所在包 = warn```
如果要保存日志信息到文件```logging.file=spring.log```就会生成spring文件名的log文件







