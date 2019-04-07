---
layout:     post
title:      "springboot practice"
subtitle:   " \"springboot practice\""
date:       2018-04-07 19:25:00
author:     "wmf"
header-img: "img/in-post/java.jpg"
catalog: true
tags:
    - java
    - spring
---
### springboot practice
##### scheduling
Enable Scheduling
```java
@SpringBootApplication
@EnableScheduling
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class);
    }
}
```
Create a scheduled task, use ```@Scheduled``` expression
The Scheduled annotation defines when a particular method runs. NOTE: This example uses fixedRate, which specifies the interval between method invocations measured from the start time of each invocation
```java
@Component
public class ScheduledTasks {
    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");

    @Scheduled(fixedRate = 5000)
    public void reportCurrentTime() {
        log.info("The time is now {}", dateFormat.format(new Date()));
    }
}
```









