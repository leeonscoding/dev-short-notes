# Example with Feign and Eureka server and spring cloud loadbalancer

## Spring boot actuator
It provides some production ready features

* send heartbeat
* app info
* health and metrics
* monitoring

## How to configure actuator
In each service/app, we need to configure the following. In our case, Employee and Address apps.

* Add spring-boot-starter-actuator in the maven or gradle dependecny
```groovy
implementation 'org.springframework.boot:spring-boot-actuator:3.1.5'
```

* Add some properties in appplication.yaml
```yaml
management:
    endpoint:
        web:
            exposure:
                include: *
    info:
        env:
            enabled: true

info:
    app:
        name: employee app
        description: simple app
        version: 1.0.0
```

Check the urls

* http://localhost:8082/employee-app/api/actuator/health
* http://localhost:8082/employee-app/api/actuator/info

etc.

# Define url and path
In doc-2 I've defined a client

```java
@FeignClient(name = "address", url = "http://localhost:3001", path = "/address/api")
```
Now, modify this as

```java
@FeignClient(name = "address", url = "ADDRESS-SERVICE", path = "/address/api")
```

Here, the url is the app name and path is the context path. We can get this in the application.yaml file located in the address app.

## Note: We don't need the spring-cloud-starter-loadbalancer dependency. Because the Feign internally includes this.

# Feign loadbalancer(internally spring-cloud-starter-loadbalancer) algorithm
Internally it uses the round-robin algorithm by default. But it is not always good. Because there is a scenario. If some users are staying longer on a specific server and using the server resource then picking a server in roun-robin fashion is inefficient.

We can use the RandomLoadBalancer.