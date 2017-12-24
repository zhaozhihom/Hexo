---
title: SpringBoot多环境配置及参数启动
date: 2016-09-22 09:05:40
categories:
tags:
    - SpringBoot
---
使用SpringBoot开发能够极大的简化配置文件，在配置不同环境时只需要一个文件即可，以下是配置文件：

<!--more-->

#### 配置文件

```yml
spring:
  datasource:
    username: root
    password: root
    driver-class-name: com.mysql.jdbc.Driver

  jpa:
    properties:
    #  hibernate.hbm2ddl.auto: create
    show-sql: true

  profiles:
    active: prod

---
spring:
  profiles: dev
  datasource:
    url: jdbc:mysql://localhost:3306/test
server:
  port: 8888
---
spring:
  profiles: prod
  datasource:
    url: jdbc:mysql://localhost:3306/server
server:
  port: 9999

```
有几点要特别注意：
1. 不同环境的配置间用`---`隔开，起到分隔作用域的效果，否则会互相冲突
2. 在主配置项spring.profiles.active：后写定义的不同配置名，如dev、prod，冒号后有一个空格


#### 同时启动多个环境
如果要同时启动多个环境，可以使用命令行的方式；
首先需要用MAVEN打包，在项目根目录下运行：`maven clean package`；
然后进入到target文件夹，启动项目
```cmd
 java -jar ***.jar --spring.profiles.active=dev
```
这样启动开发环境，同理可以启动生产环境；
也可以传递多个参数，像这样：
```cmd
 java -jar ***.jar --spring.profiles.active=prod --server.port=9090
```
SpringBoot是如何实现这样灵活的启动方式的呢，我们看一下SpringBoot的启动类里的main方法：
```Java
public static void main(String[] args) {
		SpringApplication.run(DemoNewApplication.class, args);
	}
```
我们可以看到SpringBoot将命令行中的参数先传到main主方法，再传入了run()方法，然后去设置启动参数，是不是很简单。

