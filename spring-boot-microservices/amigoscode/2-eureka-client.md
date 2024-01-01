# Steps
1. Create an application for the service
2. include dependencies in pom.xml or build.gradle
3. Annotate @EnableEurekaClient in the main class or config
4. configure application.properties or application.yaml

# Impl
2. spring-clould-starter-netflix-eureka-client

3.
```java
@SpringBootApplication
@EnableEurekaClient
public class CustomerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

4.
```yaml
server:
  port: 3000
eureka:
  client:
    service-url:
        default-zone: http://localhost:8761/eureka
spring:
  application:
    name: eureka-server
```

## Verify
In eureka server dashboard, we will see this app instance

## Connect with the service
In eureka server dashboard, take the application name.

In rest template use this way,
```java
restTemplate.getForObject("http://CUSTOMER/api/v1/customer/{id}", CustomerResponse.class, customer.getId());
```

## Load balanced
Use the @LoadBalanced annotation where the rest template bean is initialized.
```java
@Configuration
public class CustomerConfig {
  @Bean
  @LoadBalanced
  public RestTemplate restTemplate() {
    return new RestTemplate();
  }
}
```

This load balancer uses round robin algorithm.

## Types
There are 2 types of load balancers
* External - The entry point
* Internal - Inside private network among applications

## Key factors
* TLS
* Certificate management
* Authentication
* High availability
* Logging
* Caching
* Path based routing

## Popular load balancers
* GCP cloud lb
* AWS elastic lb
* NGINX

## Load balancing algorithms
* round robin
* least connections
* least time
* hash
* ip hash
* random