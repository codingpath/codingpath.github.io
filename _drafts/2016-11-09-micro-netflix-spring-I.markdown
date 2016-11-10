---
layout: post
title:  "Spring Microservices: Spring Boot + Spring Cloud + Eureka (Part I)"
date:   2016-11-09 00:00:00
categories: microservices spring netflix cloud
---
I am not going to describe the Microservice theory or the history of this architecture. I will assume that, as me, you are already hype up about this design and want to start building something with your own hands.

Here you can find the [Source Code][repo-dev] of the entire project. In this post we will use the `I_discovery` branch.

## Discovery Server

You will need something to discover and register the services available in your Microservice ecosystem. For this, we use [Eureka][eureka-page], a framework from Netflix that helps you keep track of the Microservices registered and as a Load Balancer in case of having a cluster.

# Maven Configuration

*  First, create a Maven project named `micro-discovery-server`.

*  The next step is to configure your project as a [Spring Boot][spring-boot-page] project adding the parent:

{% highlight xml %}
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.4.1.RELEASE</version>
</parent>
{% endhighlight %}

*  Once you have the parent configuration, you can start adding the dependencies needed for a WEB application:

{% highlight xml %}
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
{% endhighlight %}

We are not going to use `Tomcat` (which is the default application server embedded in `spring-boot-starter-web`), so we need exclude that dependency and add `Jetty` instead:

{% highlight xml %}
<!-- Spring Web -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!-- Replaces Default Tomcat -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
{% endhighlight %}
    
You can use `Tomcat` if you want just by leaving the dependency like the first fragment of code.

* Now that we have a [Spring Boot][spring-boot-page] project, we need to include the [Spring Cloud][spring-cloud-page] libraries in order to use [Eureka][eureka-page] as our Discovery Server. The first step to do this is adding the `dependencyManagement` for the Spring Cloud dependencies:

{% highlight xml %}
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Camden.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
{% endhighlight %}

And finally the dependencies for Eureka and Spring Cloud:

{% highlight xml %}
<!--  Spring Cloud -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter</artifactId>
</dependency>
		
<!-- Netflix Eureka Server -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
{% endhighlight %}

# Java Configuration

Thanks to Spring Boot, not just the dependency management is easier but also the Spring configuration in our Java code. In order to start our application, we need to create a `main` class:

{% highlight java %}
@SpringBootApplication
@EnableEurekaServer
public class DiscoveryServerConfig {
    
    public static void main(String[] args) throws Exception {
        SpringApplication.run(DiscoveryServerConfig.class, args);
    }
	
}
{% endhighlight %}

1.  `@SpringBootApplication` annotation: This will mark this class as the main configuration class and the execution point of our application.
2.  `@EnableEurekaServer` annotation: This indicates that our application is a Service Discovery Server and is configured to use Eureka.
3.  `SpringApplication.run(DiscoveryServerConfig.class, args);`: The Spring context is created and started.

# application.yml

{% highlight yaml %}
server:
  port: 9000

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
{% endhighlight %}

## Greetings Microservice (Expose)

# Maven Configuration
{% highlight xml %}
<!-- Rest Services -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
{% endhighlight %}

{% highlight xml %}
<!-- Netflix Eureka Client -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
{% endhighlight %}

# Java Configuration
{% highlight java %}
@SpringBootApplication
@EnableEurekaClient
public class GreetingsServiceConfig {

	public static void main(String[] args) throws Exception {
        SpringApplication.run(GreetingsServiceConfig.class, args);
    }
	
}
{% endhighlight %}

# application.yml
{% highlight yaml %}
server:
  port: 8082
  
spring:
  application:
    name: micro-client-service
    
eureka:
  instance:
    nonSecurePort: ${server.port}
  client:
    serviceUrl:
      defaultZone: ${eureka.server.url:http://localhost:9000/eureka/}
{% endhighlight %}

[repo-dev]:    https://github.com/IngJmCaicedo/micro-spring-netflix
[eureka-page]: https://github.com/Netflix/eureka/wiki
[spring-boot-page]: https://projects.spring.io/spring-boot/
[spring-cloud-page]: http://projects.spring.io/spring-cloud/
