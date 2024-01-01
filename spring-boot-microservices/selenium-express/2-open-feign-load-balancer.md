Feign client is created and was used by the netflix. They share this to the open source project netflix oss. In open source its name become OpenFeign.

Spring adopt the feign and add extra features which is added in the spring cloud.

Now Spring cloud feign is as much reliable that the netflix uses this now a days.

# Configuration
* Add proper version and dependencies in the build.gradle file.
```groovy
ext {
	set('springCloudVersion', "2022.0.4")
}

dependencies {
	implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'
    //other dependencies
}

dependencyManagement {
	imports {
		mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
	}
}
```
I've used the dependencyManagement as the spring cloud is a sub project and not linked with the spring boot project. Here I've specified the spring cloud version.

# Create the client
* Create an interface followed by the remote application. In our case it is AddressClient.java.

# Build a Feign client
* Annotate with @FeignClient
* Use the standard spring mapping with abstract methods
```java
@FeignClient(name = "address-app", url = "http://localhost:3001/address/api")
public interface AddressClient {
    @GetMapping
    String getAddress();
}
``` 
* We must provide the name or value and also the url in the annotation.

* This is like standard spring RestController, we can use the @PathVariable too.
```java
@FeignClient(name = "address-app", url = "http://localhost:3001/address/api")
public interface AddressClient {
    @GetMapping("/address/{id}")
    String getAddress(@PathVariable int id);
}
```

This interface will work as a proxy class. Spring will internally create the implementation class for this interface at runtime for us. But we need to enable package scanning for finding the client interfaces and generate implementations in runtime. We need the @EnableFeignClients annotation in the main class to do this.
```java
@SpringBootApplication
@EnableFeignClients
public class EmployeeApplication {
	public static void main(String[] args) {
		SpringApplication.run(EmployeeApplication.class, args);
	}

}
```
We can also define base packages.

If we need the full response, then the return type
```java
    @GetMapping("/address/{id}")
    ResponseEntity<AddressResponse> getAddress(@PathVariable int id);
```
We can also separate the context path too
```java
@FeignClient(name = "address", url = "http://localhost:3001", path = "/address/api")
```

The best practice for the FeignClient name is the application name. This will help in service discovery in future.

Feign client is easily integrated with other spring cloud projects like Eureka, Gateway, LoadBalancer etc.

# Load balance
Now think there are several Address service instances are running. Here we hard-coded the url(http://localhost:3001). Here, there is no way to distribute the load even if there are several instances are running. All requests will go to that url(http://localhost:3001).

Now we need to distribute the load. 

Here is a client-server relationship. The employee service requests some data to the address service. So, the employee service is the client and the address service is the server.

We can set a client side load balancer in the employee app. This LB will help us to redirect to a appropriate server instance. In legacy way, there was an api called Netflix Ribbon. Now it is depricated and removed too. Recommendend way is the Spring cloud LB.

In Ribbon, the LB works using the Round Robin algorithm. Sequentially each instances.

For example, We have 2 address instances. One is in the port 8081 and other in port 8082. We made 4 requests from the employee app sequentially. A possible sequence is
* 8082
* 8081
* 8082
* 8081

In this way, we again hard coded the instances url. If we add more instance, we can't add it in runtime. To overcome this issue we need the Service Discovery.