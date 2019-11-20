# SpringCloudEureka
SpringCloud之Eureka搭建高可用的注册中心
---
&emsp;&emsp;**第一次接触SpringCloud，在这里简单介绍一下SpringCloud及Eureka，同时记录一下使用Eureka搭建一个高可用的服务注册于发现模块。戳[这里](https://github.com/butalways1121/SpringCloudEureka)下载源码。**
<!-- more -->
## 一、SpringCloud简介
### 1.什么是SpringCloud
&emsp;&emsp;首先，尽管Spring Cloud带有“Cloud”这个单词，但它并不是云计算解决方案，它是一系列框架的有序集合，是在SpringBoot的基础上构建的。SpringCloud利用Spring Boot的开发便利性巧妙地简化了分布式系统基础设施的开发，为开发人员提供了快速建立分布式系统中的一些常见的模式，如配置管理（configuration management），服务发现（service discovery），断路器（circuit breakers），智能路由（ intelligent routing），微代理（micro-proxy），控制总线（control bus），一次性令牌（ one-time okens），全局锁（global locks），领导选举（leadership election），分布式会话(distributed sessions），集群状态（cluster state)等等。Spring Cloud并没有重复制造轮子，它只是将目前各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过Spring Boot风格进行再封装屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包。
### 2.SpringCloud的组成
&emsp;&emsp;Spring Cloud的子项目，大致可分成两类，一类是对现有成熟框架”Spring Boot化”的封装和抽象，也是数量最多的项目；第二类是开发了一部分分布式系统的基础设施的实现，如Spring Cloud Stream扮演的就是kafka、ActiveMQ这样的角色。对于想快速实践微服务的开发者来说，第一类子项目就已经足够使用，如：
&emsp;&emsp;（1）Spring Cloud Netflix：是对Netflix开发的一套分布式服务框架的封装，包括服务的发现和注册，负载均衡、断路器、REST客户端、请求路由等；
&emsp;&emsp;（2）Spring Cloud Config：将配置信息中央化保存, 配置Spring Cloud Bus可以实现动态修改配置文件；
&emsp;&emsp;（3）Spring Cloud Bus：分布式消息队列，是对Kafka, MQ的封装；
&emsp;&emsp;（4）Spring Cloud Security：对Spring Security的封装，并能配合Netflix使用；
&emsp;&emsp;（5）Spring Cloud Zookeeper：对Zookeeper的封装，使之能配置其它Spring Cloud的子项目使用；
&emsp;&emsp;（6）Spring Cloud Eureka：是 Spring Cloud Netflix 微服务套件中的一部分，它基于Netflix Eureka 做了二次封装，主要负责完成微服务架构中的服务治理功能。
### 3.SpringCloud的特点
* 约定大于配置
* 适用于各种环境
* 开箱即用
* 组件丰富，功能齐全
* 轻量级的组件
* ......

## 二、SpringCloud Eureka
### 1.Eureka简介
&emsp;&emsp;Eureka是Netflix开源的一款提供服务注册和发现的产品，是Netflix开发的服务发现框架，它提供了完整的Service Registry和Service Discovery实现。Eureka主要用于定位运行在AWS域中的中间层服务，以达到负载均衡和中间层服务故障转移的目的。
&emsp;&emsp;Eureka包含两个组件：Eureka Server和Eureka Client。其中，Eureka Server提供服务注册服务，各个节点启动后，会在Eureka Server中进行注册，这样Eureka Server中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观的看到。Eureka Client是一个java客户端，用于简化与Eureka Server的交互，客户端同时也就是一个内置的、使用轮询(round-robin)负载算法的负载均衡器。
### 2.简单实现服务注册和发现
#### （1）服务端
&emsp;&emsp;新建一个Maven Project，将pom.xml内容修改如下：
```bash
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>1.0.0</groupId>
	<artifactId>springcloud-eureka</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>
	<name>springcloud-eureka</name>
	<url>http://maven.apache.org</url>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.9.RELEASE</version>
		<relativePath />
	</parent>
	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<spring-cloud.version>Dalston.RELEASE</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka-server</artifactId>
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
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```
**需要一提的是若SpringBoot的版本为1.x，则SpringCloud的依赖版本为`<spring-cloud.version>Dalston.RELEASE</spring-cloud.version>`;若SpringBoot的版本为2.x，SpringCloud的依赖版本则要为`
<spring-cloud.version>Finchley.RELEASE</spring-cloud.version>`，而且jdk的版本必须是1.8以上。**
&emsp;&emsp;接着，在src/main下创建resources文件夹，再创建application.properities文件，并将服务端的配置信息添加进去，如下：
```bash
#指定服务名称
spring.application.name=springcloud-eureka-client
#指定服务的端口
server.port=8000
#eureka.client.register-with-eureka表示是否将自己注册到Eureka，默认是true
eureka.client.register-with-eureka=false
#eureka.client.fetch-registry表示是否从Eureka Server获取注册信息，默认是true
eureka.client.fetch-registry=false
#eureka.client.serviceUrl.defaultZone设置与Eureka Server交互的地址，客户端的查询服务和注册服务都需要依赖这个地址
eureka.client.serviceUrl.defaultZone=http://localhost:8000/eureka/
```
&emsp;&emsp;再修改启动类，这里只需添加`@EnableEurekaServer`注解即可，该注解表示此服务的是一个服务注册中心服务。代码如下：
```bash
@EnableEurekaServer
@SpringBootApplication
public class App 
{
    public static void main( String[] args )
    {
    	SpringApplication.run(App.class, args);
        System.out.println("注册中心服务启动...");
    }
}
```
#### （2）客户端
&emsp;&emsp;同样新建一个Maven Project，添加同服务端相同的依赖，然后在application.properties中添加如下配置信息：
```bash
spring.application.name=springcloud-eureka-client
server.port=9001
eureka.client.serviceUrl.defaultZone=http://server3:8003/eureka/
```
&emsp;&emsp;接着修改启动类，也是只需添加`@EnableDiscoveryClient`注解，表示该项目就具有了服务注册的功能，示例如下：
```bash
@EnableEurekaClient
@SpringBootApplication
public class App 
{
    public static void main( String[] args )
    {
    	SpringApplication.run(App.class, args);
        System.out.println( "客户端服务启动..." );
    }
}
```
#### （3）测试
&emsp;&emsp;以上两个项目完成后，依次启动服务端和客户端程序，在浏览器中输入`http://localhost:8000/`，即可查看注册中心的信息如下图，可以看到Eureka启动成功了，并且有一个服务已经注册了：
![](https://raw.githubusercontent.com/butalways1121/img-Blog/master/72.png)
## 三、搭建高可用的注册中心
&emsp;&emsp;上述的服务注册中心示例是单点的，但在生产环境中像注册中心这么关键的服务，如果是单点话，遇到故障就是毁灭性的。在一个分布式系统中，服务注册中心是最重要的基础部分，理应随时处于可以提供服务的状态。为了维持其可用性，使用集群是很好的解决方案，Eureka通过互相注册的方式来实现高可用的部署。接下来再新建两个服务端的工程，让它们互相注册，从而搭建一个高可用的注册中心。
### 1.新建服务端工程server2
&emsp;&emsp;同样新建一个Maven Project，配置中稍有不同的是这里的配置允许自己进行注册，即将上述配置中的`eureka.client.register-with-eureka=false`去掉或者改为true即可，如下：
```bash
spring.application.name=springcloud-eureka-client
server.port=8002
eureka.client.serviceUrl.defaultZone=http://server3:8003/eureka/
eureka.instance.hostname = server2
```
这里的`eureka.instance.hostname= server2`相当于对服务地址起别名，以便于区分服务，因为起了别名，所以需要在本机`C:\Windows\System32\drivers\etc\hosts`文件中添加如下配置，做映射：
```bash
127.0.0.1     server2
127.0.0.1     server3
```
### 2.新建服务端工程server3
&emsp;&emsp;同server2，配置如下：
```bash
spring.application.name=springcloud-eureka-client
server.port=8003
eureka.client.serviceUrl.defaultZone=http://server2:8002/eureka/
eureka.instance.hostname = server3
```
### 3.修改客户端工程
&emsp;&emsp;将之前的客户端程序配置application.properities文件中`eureka.client.serviceUrl.defaultZone=http://server3:8003/eureka/`修改为`eureka.client.serviceUrl.defaultZone=http://server3:8003/eureka/`，使其指向新建的高注册可用中心两个服务中的一个即可。
### 4.测试
&emsp;&emsp;首先，启动server2和server3两个服务，在浏览器中输入`http://server2:8002/`或`http://server3:8003/`即可查看信息，如下图，可以看到两个服务已经相互注册了：
![](https://raw.githubusercontent.com/butalways1121/img-Blog/master/73.png)
&emsp;&emsp;接着，启动配置修改好的客户端，再关掉其指向的那个服务，即终止server3服务，在浏览器输入`http://server2:8002/`查看该服务是否正常运行，如下图：
![](https://raw.githubusercontent.com/butalways1121/img-Blog/master/74.png)若结果如图，则一个简单的高可用的注册中心项目已经搭建完成！
***
&emsp;&emsp;**总体来讲，这个小项目确实很简单，需要注意的就是文中提到的SpringCloud的依赖版本，另外，若想让其他接口也注册到Eureka，也十分简单，首先在pom.xml文件中加入相关依赖，接着在application.properities中添加服务端的配置信息，接着在接口类中添加相应注解即可。**
