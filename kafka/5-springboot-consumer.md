## Tell consumer about running bootstrap server
```yaml
server:
  port: 3002
spring:
  kafka:
    consumer:
      bootstrap-servers: "localhost:9092"
      group-id: grp1
```
Here consumer group-id isn't mandatory

## Create a service class which will listen the messages. Must define topic name and consumer group id.
```java
package com.leeonscoding.kafkaconsumer101.service;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

@Service
public class TestMessageListenerService {

    private static final String TOPIC_NAME = "test-topic1";
    private static final String CONSUMER_GROUP_ID = "grp1";

    private Logger log = LoggerFactory.getLogger(TestMessageListenerService.class);

    @KafkaListener(topics = TOPIC_NAME, groupId = CONSUMER_GROUP_ID)
    public void consume(String message) {
        log.info(message);
    }
}
```

# N.B: If there are 4 groups of a same topic and 2 partitions of the topic then only 2 groups will receive the messages. which groups will receieve that is managed by zookeeper.

# N.B: In real life apps, don't write multiple consumer group instances of a same topic. We can do it using concurrency.

# While consuming, if the consumer applicaiton shutdown, there is a possibility some data isn't consumed. In that case
* lag = data count which isn't consumed
* offset = data count which is consumed