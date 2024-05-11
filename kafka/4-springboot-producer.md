## application.properties/application.yaml configuration
```yaml
server:
  port: 3001
spring:
  kafka:
    producer:
      bootstrap-servers: "localhost:9092"
```

## create a new topic with partition and replication factor if needed. create a package config and add a java class called KafkaConfig
```java
package com.leeonscoding.kafkaproducer101.config;

import org.apache.kafka.clients.admin.NewTopic;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class KafkaConfig {

    @Bean
    public NewTopic createTest3Topic() {
        return new NewTopic("test3", 3, Integer.valueOf(1).shortValue());
    }

}

```

## create a service class which will interact with kafka directly.
```java
package com.leeonscoding.kafkaproducer101.service;

import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.SendResult;
import org.springframework.stereotype.Service;

import java.util.concurrent.CompletableFuture;

@Service
public class MessagePublisherService {
    private KafkaTemplate<String,Object> template;

    public MessagePublisherService(KafkaTemplate<String, Object> template) {
        this.template = template;
    }

    public void sendMessageToTopic(String message) {
        CompletableFuture<SendResult<String, Object>> result = template.send("test-topic1", message);

        result.whenComplete((res, exp) -> {
            if(exp == null) {
                System.out.println("message sent successfully");
            } else {
                System.out.println("an exception occurred");
            }
        });
    }
}
```

## create a controller class which will interact with the service class

```java
package com.leeonscoding.kafkaproducer101.controller;

import com.leeonscoding.kafkaproducer101.service.MessagePublisherService;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/producer")
public class EventController {
    private MessagePublisherService messagePublisherService;

    public EventController(MessagePublisherService messagePublisherService) {
        this.messagePublisherService = messagePublisherService;
    }

    @GetMapping("/{message}")
    public ResponseEntity<?> publishMessage(@PathVariable String message) {
        try {
            messagePublisherService.sendMessageToTopic(message);

            return ResponseEntity.ok("message is processing");
        } catch (Exception e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).build();
        }

    }
}
```