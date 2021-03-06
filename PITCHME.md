
# Spring Cloud 


---

## Agenda

* Overview Spring Cloud 
* Config Server 
* Service Registry
* Resilience
* References
* Exercises

---

## Sources

 * https://cloud.spring.io
 * https://12factor.net
 * slideshare
 * github
 * See References & Reading List

---

## Taxonomy

- Spring Boot
- Spring Cloud
- Spring Cloud Netflix
- Spring Cloud Services

+++

# Spring Cloud

- based on Spring Boot
- provides tools for developers to quickly build apps
- addresses common patterns in distributed systems
- not focused on a particular platform

![Spring Cloud](http://cdn.springtutorials.com/wp-content/uploads/2015/10/spring-cloud.png)

+++

Review

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">For those on <a href="http://t.co/uB4AvizyvG">http://t.co/uB4AvizyvG</a> saying it’s Spring Boot is “the framework for a framework” here’s a diagram: <a href="http://t.co/5LvwSiWKo0">pic.twitter.com/5LvwSiWKo0</a></p>&mdash; Phil Webb 🍃 (@phillip_webb) <a href="https://twitter.com/phillip_webb/status/641444531867680768?ref_src=twsrc%5Etfw">September 9, 2015</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

+++

Spring Framework

![Spring Framework Cupcake](https://pbs.twimg.com/media/CObcO3gUEAAKV6Y.jpg)

+++

Spring Boot

![Spring Framework Cupcake](https://pbs.twimg.com/media/CObc-WbWoAEPaSz.jpg)

+++

Spring Cloud

![Spring Cloud Cupcake](https://pbs.twimg.com/media/CObf6AQWEAIMkAS.jpg)

+++

## Patterns addressed

- configuration management
- service registry and discovery
- resilience and circuit breakers
- intelligent routing and load balancing
- and more ...

+++

![Spring Cloud](https://spring.io/img/homepage/diagram-distributed-systems.svg)


+++

## Spring Cloud Netflix

- originated as Netflix OSS
- based on Spring implementations
- partially integrated into Spring Cloud projects
- now a core project of Spring Cloud

+++ 

## Services Provided

- Eureka - service registry and discovery
- Hystrix - circuit breaker and dashboard
- Feign - declarative REST client
- Ribbon - cient-side load balancing
- Zuul - routing & filtering

+++

## Spring Cloud Services

![SCS](https://3.bp.blogspot.com/-nvGF74M_hyY/V1zWlx_0RLI/AAAAAAAAPL0/CPwG4Azzpc4b9Kx2LhHkAxdkpqxkwLtbgCLcB/s1600/SpringCloudServices.png)

+++

## Links

- https://projects.spring.io/spring-boot/
- http://projects.spring.io/spring-cloud/
- http://netflix.github.io/
- https://docs.pivotal.io/spring-cloud-services/1-3/common/

---

# Config Server

+++

### Review twelve-factor app

# III. Config

+++

### Store config in the environment

* An app’s config is everything that is likely to vary between deploys (staging, production, developer environments, etc). 
* This includes:
  - Resource handles to the database, Memcached, and other backing services
  - Credentials to external services such as Amazon S3 or Twitter
  - Per-deploy values such as the canonical hostname for the deploy

+++

## Rules

* Apps sometimes store config as constants in the code. 
* This is a violation of twelve-factor, which requires strict separation of config from code. 
* Config varies substantially across deploys, code does not.

+++

### External configuration by file

- Can be injected through a property file
- Prior to Spring Boot the config file had to be referenced

```java
@PropertySource("file:/path/to/simple.properties")
public class Application {
```

```java
    @Value("${spring.application.name}")
    private String appName;
```
+++

### External configuration by file

- Properties can be set with a default value, which will be picked in case property is not found or set
- In case no default is defined and the property can't be found the app won't start

```java
    @Value("${spring.application.name: Default Name}")
    private String appName;
```
```
2017-05-03 13:55:26.342 [..] Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'messageRestController': Injection of autowired dependencies failed; nested exception is java.lang.IllegalArgumentException: Could not resolve placeholder 'spring.application.name' in value "${spring.application.name}"
```
+++

### External configuration by file with Spring Boot

- Spring Boot will read the properties in `src/main/resources/application.properties` by default
- Spring Boot allows you to use yaml files instead of properties
- Both examples will work

```yaml
application.name=a-bootiful-client
```
```yaml
application:
    name: a-bootiful-client
```
+++

### External configuration by file with Spring Boot- POJO

```java
@Component
@ConfigurationProperties("application")
class ApplicationProperties{

	private String name;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

}
```

+++

### External configuration by file with Spring Boot- POJO Invocation

```java
@RestController
class MessageRestController {

    @Autowired
    private ApplicationProperties ap;

    @RequestMapping("/appNameViaPOJO")
    String appNameViaPOJO() {
        return this.ap.getName();
    }
}
```

+++

# Remember Actuator?

+++

Point the browser to: http://localhost:8080/env

```json
"applicationProperties": {
  "prefix": "application",
  "properties": {
    "name": "a-bootiful-client"
  }
},
```

+++

### Limitations of current approach

- restart required to change the property files
- no traceability of changes
- de-centralized configuration
- no ootb security/encryption mechanism

+++

## Config Server 
### to address limitations

+++

- provides dynamic properties to the clients during runtime
- requires a git repository (online or file based)
- serves as proxy for configuration keys and values 
- http://cloud.spring.io/spring-cloud-config/

+++

![Config_Server](https://docs.pivotal.io/spring-cloud-services/1-0/images/config-server/config-server-fig1.png)

+++

## Java

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConfigServerApplication.class, args);
	}
}
```
+++

## application.yml

```yaml
server:
  port: 8888

spring:
  cloud:
    config:
      server:
        git :
          uri: https://github.com/maeddes/config-server
```

+++

My GitHub
<iframe data-src="https://github.com/maeddes/config-server" height="480px" width="100%"></iframe>

+++

## Validation
http://localhost:8888/config-client/master

```json
{
  "name": "config-client",
  "profiles": [
    "master"
  ],
  "label": null,
  "version": null,
  "state": null,
  "propertySources": [
    {
      "name": "https://github.com/maeddes/config-server/config-client.properties",
      "source": {
        "config.property": "43"
      }
    }
  ]
}
```
+++

# Config client

+++

## Java

```java
@RestController
@RefreshScope
class ConfigServerRestController{
	
	@Value("${config.property: Default}")
	private String configProperty;

	@RequestMapping("/valueFromServer")
	String valueFromServer() {
		return this.configProperty;
	}
	
	
}
```

+++

## bootstrap.yml

```yaml
spring:
  application:
    name: config-client
    cloud:
      config:
        uri: http://localhost:8888
```
+++

## Updates

- To make a runtime change you need to update it in the git repo first
- You can validate with the config server URL, if it has been picked up
- The clients will not update automatically
- It requires a POST action to the http://127.0.0.1:8080/refresh endpoint

```yaml
mhs@R2-D2:~$ curl -X POST http://127.0.0.1:8080/refresh
["config.property"]mhs@R2-D2:~$ 
```

+++

# start.spring.io

Config Server
- Config Server

Config Client
- Config Server

---

# Service Registry

+++

### Review Microservices

- Microservice are meant to run independent
- Mulitple services are allowed to run distributed and in parallel
- To enable communication the services need to know how to find each other
- Manual configuration can be a hell of a task

+++ 

> Microservices is defined as a loosely-coupled, service-oriented architecture with bounded context.

Adrian Cockroft, 2014

+++

### High - Level Architecture

![Eureka](https://docs.pivotal.io/spring-cloud-services/1-2/images/service-registry/Netflix-Eureka-d1.png)

+++

### Eureka Server

+++

### Java

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@EnableEurekaServer
@SpringBootApplication
public class ServiceEurekaApplication {

	public static void main(String[] args) {
		SpringApplication.run(ServiceEurekaApplication.class, args);
	}
}
```

+++

### application.yml

```yaml
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

management:
  security:
    enabled: false
```
+++

# Eureka Client

- Both service provider and service consumer register at Eureka server as client
- The server will register them by the property spring.application.name

+++

### Eureka service/client

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class MhsEurekaServiceApplication {

	public static void main(String[] args) {
		SpringApplication.run(MhsEurekaServiceApplication.class, args);
	}
}

...
}
```
+++

### Exposed service

```java
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;


@RestController
class ConfigFileRestController {
	
    @RequestMapping("/service")
    String service() {
        return "Service exposed to Eureka";
    }
}
```

+++

### bootstrap.yml

```yaml
spring:
  application:
    name: mhs-service
```

+++

### application.yml

```yaml
server:
  port: 0
```

- The service will be bound to a random port
- Port information will not be exposed to client directly
- Application is supposed to be lookup up by id only

+++

### application.yml with multiple instance support

```yaml
server:
  port: 0
  
eureka:
  instance:
    leaseRenewalIntervalInSeconds: 5
    instanceId: ${spring.application.name}:${random.value}
```

- Eureka will only allow one instance with a given id
- A random value needs to be generated and appended

+++

# Service Consumer

+++

### Java Header part

```java

import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
...

@EnableDiscoveryClient
@SpringBootApplication
public class MhsEurekaConsumerApplication {

	public static void main(String[] args) {
		SpringApplication.run(MhsEurekaConsumerApplication.class, args);
	}
}
```

+++

### Java functional part

```java
@RestController
class ServiceInstanceRestController{

    @Autowired
    private DiscoveryClient discoveryClient;

    @RequestMapping("/serviceList")
    public String serviceList(){
            
        return this.discoveryClient.getServices().toString();
    }

    @RequestMapping("/service-instances/{applicationName}")
    public List<ServiceInstance> serviceInstancesByApplicationName(
            @PathVariable String applicationName) {
        return this.discoveryClient.getInstances(applicationName);
    }
    
}
```

+++

### bootstrap.yml

```yaml
spring:
  application:
    name: service-consumer
```

- can also be empty or non-existent
- client does not need to be looked up
- better style still ;-)

+++

## start.spring.io


- Eureka Server
- Eureka Discovery
- Web


---

# service to service communication

+++

Traditional way using URL

```java
@Autowired
private EurekaClient discoveryClient;

public String serviceUrl() {
    InstanceInfo instance = discoveryClient.getNextServerFromEureka("mhs-service", false);
    return instance.getHomePageUrl();
}
```

+++

```java
@Bean
public RestTemplate restTemplate() {
	return new RestTemplate();
}

@Autowired
private RestTemplate rest;

@RequestMapping("/serviceViaTemplate")
public String serviceInvocationRT(){
    String str = rest.getForObject(serviceUrl(), String.class);
}
```

+++
## Direct invocations of Eureka service name

- Ideally you should not have to deal with the URLs directly
- Spring Cloud provides two improved alternatives
  - Ribbon + REST Template
  - Feign
  
+++

## Ribbon

- Ribbon is a client side IPC library that is battle-tested in cloud
- Provides client-side load balancing and fault tolerance
  
+++

```java
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
	return new RestTemplate();
}
   
@Autowired
private RestTemplate rest;

@RequestMapping("/serviceViaTemplate")
public String serviceInvocation(){
        return this.rest.getForObject("//mhs-service/service", String.class);
}
```
+++

## Feign

- A declarative REST client
- To simplify HTTP client calls
- Only requires declaration and annotation of an interface while the actual implementation will be provisioned at runtime

+++

```java
@EnableFeignClients
public class EurekaConsumerApplication 
...

@FeignClient("mhs-service")
interface ServiceClient{
	@RequestMapping(value = "/service", method = RequestMethod.GET)
	String service();
}

@Autowired
private ServiceClient serviceClient;

@RequestMapping("/serviceViaFeign")
public String serviceInvocation(){
   	return this.serviceClient.service();
}

```
+++

## start.spring.io

- Ribbon
- Feign

---

# Resilience

+++

![Hystrix](https://netflix.github.com/Hystrix/images/hystrix-logo-tagline-850.png)

+++

#Hystrix 

- Hystrix Dashboard
- Hystrix Stream
- Circuit Breaker

+++

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;

@SpringBootApplication
@EnableHystrixDashboard
public class MhsHystrixDashboardApplication {

	public static void main(String[] args) {
		SpringApplication.run(MhsHystrixDashboardApplication.class, args);
	}
}
```

+++

http://localhost:8080/hystrix
![Dashboard](img/hystrix_login.jpg)

+++

http://localhost:8080/hystrix
![Failing](http://cloud.spring.io/spring-cloud-static/Camden.SR5/images/HystrixFallback.png)

+++

### Failing Service

```java
@Service 
class PotentiallyFailingService{
	
	public int getNumber() throws Exception{
		if (Math.random() > .5){
			Thread.sleep(1000);
			throw new RuntimeException("Service Failed!");
		}
		return 1;
	}
	
}
```

+++
### Failing Service with Fallback

```java
@Service 
class PotentiallyFailingService{
	
	public int fallback(){  // requires same signature as getNumber
		return 2;
	}
	
	@HystrixCommand(fallbackMethod = "fallback")
	public int getNumber() throws Exception{
		if (Math.random() > .5){
			Thread.sleep(1000);
			throw new RuntimeException("Service Failed!");
		}
		return 1;
	}
	
}
```

+++

### REST Controller

```java
@RestController
class ConfigFileRestController {
	
    private final PotentialFailingService potentialFailingService;
	
    @Autowired
    public ConfigFileRestController(PotentialFailingService potentialFailingService) {
		this.potentialFailingService = potentialFailingService;
	}
    
    @RequestMapping("/service")
    String service() throws Exception {
    	return "Number service: "+this.potentialFailingService.getNumber();
    }
}
```

+++

### Main Application

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.stereotype.Service;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;

@SpringBootApplication
@EnableCircuitBreaker
public class MhsCircuitBreakerApp {

	public static void main(String[] args) {
		SpringApplication.run(MhsHystrixDashboardApplication.class, args);
	}
}
```

+++

## Hystrix Stream

- Requires enabled features Actuator and Hystrix
- Will expose a Stream at `http://<host>:<port>/hystrix.stream`
- Stream can then be imported to Hystrix Dashboard

+++

## Stream

![Hystrix Stream](img/hystrix_stream.jpg)

+++

## Dashboard

![Hystrix Dashboard](img/hystrix_dashboard.jpg)

+++

# start.spring.io

- Hystrix Dashboard
- Hystrix
- Actuator (for Stream)

---

# Reading List

+++

Part 1:

- https://spring.io/blog/2015/01/20/microservice-registration-and-discovery-with-spring-cloud-and-netflix-s-eureka
- https://docs.pivotal.io/spring-cloud-services/1-3/common/service-registry/writing-client-applications.html
- https://spring.io/guides/gs/service-registration-and-discovery/
- https://www.slideshare.net/AndreasFalk2/cloud-foundry-meetup-stuttgart-2017-spring-cloud-development
- https://www.innoq.com/de/articles/2016/12/microservices-a-la-netflix/
- http://cloud.spring.io/spring-cloud-netflix/
- http://cloud.spring.io/spring-cloud-static/Dalston.RELEASE/

+++

Part 2: 

- http://callistaenterprise.se/blogg/teknik/2015/04/10/building-microservices-with-spring-cloud-and-netflix-oss-part-1/
- http://cloud.spring.io/spring-cloud-netflix/spring-cloud-netflix.html
- http://stackoverflow.com/questions/31976236/whats-the-difference-between-enableeurekaclient-and-enablediscoveryclient
- http://www.baeldung.com/spring-cloud-netflix-eureka
- http://www.baeldung.com/intro-to-feign
- http://ryanjbaxter.com/2015/10/12/building-cloud-native-apps-with-spring-part-5/
- https://dzone.com/articles/spring-cloud-netflix-load-balancer-with-ribbonfeig

---

# Exercises I

+++

- Build a simple "Hello, World" Micro Service, which exposes the service via REST
- Implement a config server that talks to a local or remote git repo
- Extend the Micro Service to receive the config from the config server
- Trigger a config update in the Micro Service
- Implement a Eureka server
- Extend the Micro Service to be registered at the Eureka Server
- Make the Micro Service scalable to multiple instances

# Exercises II

+++

- Implement a Eureka client to discover available Micro Services
- Extend the Eureka client to invoke the service of your Micro Service
- Implement a Hystrix Dashboard
- Extend the Micro Service to have a fallback method to the service method
- Display the Micro Service in the Hystrix Dashboard
