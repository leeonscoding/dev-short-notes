# Config and run
* Eureka server in http://localhost:5000
* Address service in http://localhost:8080
* Employee service in http://localhost:8082

## Note: don't hardcode url in microservice config

For example, in the employee service app, We've hardcoded the address service url[part-1: 1-starting.md].
```yaml
server:
  port: 3002
  servlet:
    context-path: /employee/api
spring:
  application:
    name: employee-app

address-service:
  base-url: http://localhost:3001/address/api
```
As in microservice architecture, the url is changing frequently. It is difficult to debug the error.

# Get address service details from eureka server
We can get the details from the discovery eureka server. So, we don't need to hardcoded the url. We have to add the DiscoveryClient class and don't need to create any bean for this. Spring will automatically create bean for this.

```java
@Autowired
private DiscoveryClient discoveryClient;
```

Now, use it like the following

```java
List<ServiceInstance> instances = discoveryClient.getInstance("address-service");
ServiceInstance instance = instances.get(0);
String uri = instance.getUri().toString();

String address = restTemplate.getForObject(uri, String.class);
```

# Using loadbalancer
If there are several instances are running of the address service, then using this code:

instances.get(0);

is inefficient. All the requests will go to the first instance only.

Here, we need a loadbalancer. Instead of DiscoveryClient, we have to use the LoadBalancerClient.
```java
@Autowired
private LoadBalancerClient loadbalancerClient;
```
Now, use it like the following

```java
ServiceInstance instance = loadbalancerClient.choose("address-service");
String uri = instance.getUri().toString();

String address = restTemplate.getForObject(uri, String.class);
```

# metadata map
When a service gets some data from another service, the calle service can share some common things using metadata.map property. For example, the context path, username etc.
```yaml
eureka:
  instance:
    meatadata-map:
      username: test1
      xyz: some-random-value
```

And use it in the caller service like this
```java
String xyz = instance.getMetadata().get("xyz");
```

# Alternate way
Instead of doing all the loadbalaced things manually, we can do it in a simple way.
* Use the @Loadbalanced annotation where the RestTemplate is initialized
```java
@Configuration
public class AppConfig {

    @Loadbalanced
    @Bean
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

And use the context-path and application name in the url. The format is

http://{application name}/{context-path}/

Here it is [see 1-starting.md]

```java
String address = restTemplate.getForObject("http://address-service/address/api", String.class);
```

The application name is case-insensetive.
