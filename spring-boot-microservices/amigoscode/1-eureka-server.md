# Steps
1. Create an application for the eureka server
2. include dependencies in pom.xml or build.gradle
3. Annotate @EnableEurekaServer in the main class or config
4. configure application.properties or application.yaml
    * add port
    * add application name
    * set fetch-registry: false
    * set register-with-eureka: false

# Impl
2. spring-clould-starter-netflix-eureka-server

3.
```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

4.
```yaml
server:
  port: 8761
eureka:
  client:
    fetchRegistry: false
    registerWithEureka: false
spring:
  application:
    name: eureka-server
```

## Access the dashboard
Use this url

http://localhost:8761