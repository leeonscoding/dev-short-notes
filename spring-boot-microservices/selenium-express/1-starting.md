# How two applications can communicate with each other

* Generate 2 applications with only web dependency. One is employee and other one is address.
* Define a controller in the address application
```java
@RestController
public class AddressController {

    @GetMapping
    public String getAddress() {
        return "House#8/5, road#4/A, Dhanmondi, Dhaka";
    }

}
```
* Change the port of the address app
```yaml
server:
  port: 3001
```
* Define a controller in the employee app
```java
@RestController
public class EmployeeController {

    @GetMapping
    public String getEmployee() {
        return "Name: Asfaq Sufian";
    }
}
```
* Change the port of the employee app
```yaml
server:
  port: 3002
```
* Check both endpoints by running both the apps
```bash
curl http://localhost:3001/
```
```bash
curl http://localhost:3002/
```
* Now the target is call the getAddress() from the address app inside the employee app using the rest call
    * We can do this using the RestTemplate which is a rest client of spring. But it is depricated
    * Alternative modern way is the WebClient. It provides efficient non-blocking async along with traditional sync blocking approach.

# Using RestTemplate
* Create the RestTemplate bean
```java
@Configuration
public class AppConfig {

    @Bean
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```
* Grab the address value from the address app GET resource
```java
@RestController
public class EmployeeController {

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping
    public String getEmployee() {

        String address = restTemplate.getForObject("http://localhost:3001/", String.class);

        return "Name: Asfaq Sufian, Address: " + address;
    }
}
```
* Check the endpoint
```bash
curl http://localhost:3002/
```

# Add some properties for simplicity
* Add base url and application name in the Address app
```yaml
server:
  port: 3001
  servlet:
    context-path: /address/api
spring:
  application:
    name: address-app
```
* Same with the employee app. Also add the address app url for reuse throughout the app
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
* Now modify the java code in employee app
```java
@RestController
public class EmployeeController {

    @Autowired
    private RestTemplate restTemplate;

    @Value("${address-service.base-url}")
    private String addressBaseUrl;

    @GetMapping
    public String getEmployee() {

        String address = restTemplate.getForObject(addressBaseUrl, String.class);

        return "Name: Asfaq Sufian, Address: " + address;
    }
}
```
* Now hit the url
```bash
curl http://localhost:3002/employee/api/
```

# Using RestTemplateBuilder
```java
@RestController
public class EmployeeController {

    private RestTemplate restTemplate;

    @Value("${address-service.base-url}")
    private String addressBaseUrl;

    public EmployeeController(RestTemplateBuilder restTemplateBuilder) {
        this.restTemplate = restTemplateBuilder.build();
    }

    @GetMapping
    public String getEmployee() {

        String address = restTemplate.getForObject(addressBaseUrl, String.class);

        return "Name: Asfaq Sufian, Address: " + address;
    }
}
```

# Problems with RestTemplate
When a user requests to a tomcat server, the server allocates a thread from the thread pool. The thread will be ideal until the request is fully processed and the response is returned. The RestTemplate gets data from a network. There could be some network latency. So that the thread will be allocated until the data gets successfully. So, the thread will not deallocate in that time.

The RestTemplate is a blocking call. Usually the remote servers response takes time due to network latency. So, if all thread in the thread pool are blocked, new requests will not be processed.

# Assign random port dynamically
```yaml
server:
  port: 0
```
Now each time we run/deploy the app, a new random port will be assigned for the app. Using this technique, we can create new instances onece other instances are full. But this will costly.

The problem is the code
```java
String address = restTemplate.getForObject(addressBaseUrl, String.class);
```
is a blocking(thread) code.

# Alternate is WebClient
This is an api included in WebFlux. This gives an async way to retrive network responses. This thing doesn't block the thread which is allocated to retrive the network object. Instead using Observer or pub-sub pattern it will be notified when the response has come. In the meantime the thread can process other requests.

Although this is the best way the Spring framework team recommends FeignClient over WebClient.