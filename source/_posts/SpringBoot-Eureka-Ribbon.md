---
title: Ribbon实现服务消费和负载均衡
date: 2017-05-21 15:37:09
categories:
tags:
    - Java
    - SpringBoot
---

接上回Eureka实现注册中心，这次我们要利用SpringBoot中Ribbon实现服务消费端以及服务端的负载均衡。

<!--more-->

1.首先，要启动注册中心，启动多个服务提供端，即上一例中的CLIENT1. 启动的方式可以通过jar的方式启动：
```cmd
java -jar ***.jar --server.port=1994
java -jar ***.jar --server.port=1995
```
2.新建一个SpringBoot项目，引入web, eureka, ribbon的依赖：
```yml
<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.6.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<spring-cloud.version>Dalston.SR3</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka-server</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-ribbon</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
```
3.在启动类上加上注册客户端注解，创建调用REST接口的restTemplate实例，@LoadBalanced用来开启负载均衡：
```Java
@EnableDiscoveryClient
@SpringBootApplication
public class DemoRibbonCustomerApplication {

	@Bean
	@LoadBalanced
	RestTemplate restTemplate(){
		return new RestTemplate();
	}

	public static void main(String[] args) {
		SpringApplication.run(DemoRibbonCustomerApplication.class, args);
	}
}
```

4.将消费端注册到注册中心
application.yml配置：
```yml
server:
  port: 2012

eureka:
  client:
    service-url:
      defaultZone: http://sever2.zzh123.com:1896/eureka/,http://server1.zzh123.com:1895/eureka/

spring:
  application:
    name: server_ribbon
```

5.编辑控制类，调用服务端接口：
```Java
@RestController
public class CustomerController {

    @Autowired
    RestTemplate restTemplate;

    @RequestMapping(value = "/ribbon-customer", method = RequestMethod.GET)
    public String helloCustomer(){
        return restTemplate.getForEntity("http://CLIENT1/",String.class).getBody();
    }

}
```
注意这里是用应用名CLIENT1来进行调用的

6.启动注册中心SERVER1895,SERVER1896，服务提供端CLIENT1(2)，服务消费端SERVER_RIBBON:

- SERVER1895,SERVER1896
![server1895][1]
![server1896][2]

- 访问服务消费端server_ribbon：
![ribbon_client][3]

可以看到调用接口成功，得到了输出hello world

可以看到消费端控制台信息：
```console
DynamicServerListLoadBalancer for client CLIENT1 initialized: DynamicServerListLoadBalancer:{NFLoadBalancer:name=CLIENT1,current list of Servers=[PC-ofzzh.lan:1896, PC-ofzzh.lan:1895],Load balancer stats=Zone stats: {defaultzone=[Zone:defaultzone;	Instance count:2;	Active connections count: 0;	Circuit breaker tripped count: 0;	Active connections per server: 0.0;]
```
检测到current list of Servers=[PC-ofzzh.lan:1896, PC-ofzzh.lan:1895]两个服务提供者，并且通过轮询达到了负载均衡的效果

  [1]: ../../../../images/server_1895_ribbon.png
  [2]: ../../../../images/server_1896_ribbon.png
  [3]: ../../../../images/client_ribbon.png