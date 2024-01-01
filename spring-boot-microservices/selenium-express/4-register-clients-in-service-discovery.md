# Configure Eureka client
Previously I've build the address and employee services. Let's start with the address app

* add the eureka discovery client in the build.gradle
```groovy
implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
```

# How eureka server and client communicates
* When these services starts, they know where eureka usually runs. It's 8761.
* So, they use the default 8761 to connect and register themselves with eureka.
## Now what if we change the port
* Now we have to let these services know where the eureka is running. For example, 5000
* If we don't do it, it will by default try to look for eureka in 8761 and the result is exception and failure.
## Define url in clients
* To overcome this 8761 lookup exception from eureka clients, we have to define a property in application.yaml.
In address and employee applications, define this property
```yaml
eureka:
    client:
        service-url:
            defaultZone: http://localhost:5000/eureka
```

If we have multiple zones, then we can define using z1,z2... etc.

# Note
If we want to add our discovery server in the registry(registerWithEureka: true), Then we must put the service url in the discovery server too like other services(address, employee).

```yaml
server:
  port: 5000
eureka:
  client:
    fetchRegistry: false
    registerWithEureka: true
    service-url:
        defaultZone: http://localhost:5000/eureka
spring:
  application:
    name: discovery-service-app
```