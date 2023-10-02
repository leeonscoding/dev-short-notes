# Add xml file

Add beans.xml under src/main/resources folder

# Add xml content
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <bean class="com.leeonscoding.beansxml.Car"></bean>
</beans>
```

# Import the xml file using @ImportResource
```java
@Configuration
@ImportResource("classpath:beans.xml")
public class XMLBeanConfig {

}
```

# Create the POJO class
```java
public class Car {
    private String name;
    private String origin;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getOrigin() {
        return origin;
    }

    public void setOrigin(String origin) {
        this.origin = origin;
    }
}

```

# Using the bean
```java
@SpringBootApplication
public class BeansXmlApplication implements CommandLineRunner {

	public static void main(String[] args) {
		SpringApplication.run(BeansXmlApplication.class, args);
	}


	@Autowired
	private Car car;

	@Override
	public void run(String... args) throws Exception {
		car.setName("toyota");
		car.setOrigin("japan");

		System.out.println("-------"+ car.getName() + " : "+car.getOrigin());
	}
}
```