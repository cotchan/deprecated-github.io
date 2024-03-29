---
title: sb3) KafkaTemplate 사용 방법
author: cotchan 
date: 2021-05-19 21:09:21 +0800 
categories: [Spring-Boot3]
tags: [kafka] 
---

+ **이 포스팅은 개인 공부 목적으로 작성한 포스팅입니다**

---

## 0. INTRO

+ **KafkaTemplate를 사용하여 `어떻게 메시지를 전파하고, 수신하는지` 알아보겠습니다.**

---

## 1. Using Java Configuration

---

## 1-1. ProducerConfig

```java
@Configuration
class KafkaProducerConfig {

  @Value("${kafka.bootstrap-servers}")
  private String bootstrapServers;

  @Bean
  public Map<String, Object> producerConfigs() {
    Map<String, Object> props = new HashMap<>();
    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,
      bootstrapServers);
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
      StringSerializer.class);
    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
      StringSerializer.class);
    return props;
  }

  @Bean
  public ProducerFactory<String, String> producerFactory() {
    return new DefaultKafkaProducerFactory<>(producerConfigs());
  }

  @Bean
  public KafkaTemplate<String, String> kafkaTemplate() {
    return new KafkaTemplate<>(producerFactory());
  }
}
```

---

## 1-2. ConsumerConfig

```java
@Configuration
class KafkaConsumerConfig {

  @Value("${kafka.bootstrap-servers}")
  private String bootstrapServers;

  @Bean
  public Map<String, Object> consumerConfigs() {
    Map<String, Object> props = new HashMap<>();
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,
      bootstrapServers);
    props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,
      StringDeserializer.class);
    return props;
  }

  @Bean
  public ConsumerFactory<String, String> consumerFactory() {
    return new DefaultKafkaConsumerFactory<>(consumerConfigs());
  }

  @Bean
  public KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<String, String>> kafkaListenerContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<String, String> factory =
      new ConcurrentKafkaListenerContainerFactory<>();
    factory.setConsumerFactory(consumerFactory());
    return factory;
  }
}
```

---

## 1-3. application.yml

```yml
kafka:
  bootstrap-servers: localhost:9092
  consumer-group: //producer랑 consumer 값이 다름
  topic:
    comment-created-topic: v1.event.comment-created

```

---

## 2. Creating Kafka Topics

```java
@Configuration
public class KafkaTopicConfig {

    @Value("${kafka.topic.comment-created-topic}")
    String commentCreatedTopic;

    @Bean
    public NewTopic commentCreatedTopic() {
        return TopicBuilder.name(commentCreatedTopic).build();
    }
}
```

---

## 3. Sending Messages

+ **`KafkaTemplate.send()` 메소드를 사용해 특정 토픽으로 메시지를 전파합니다.**

```java
//CommentCreateEventListener.java

public class CommentCreateEventListener implements AutoCloseable {

  @Value("${kafka.topic.comment-created-topic}")
  String commentCreatedTopic;

  private KafkaTemplate<String, String> kafkaTemplate;
  
  //...

  @Autowired
  CommentCreateEventListener(KafkaTemplate<String, String> kafkaTemplate, ...) {
    this.kafkaTemplate = kafkaTemplate;
    ...
  }

  @Subscribe
  public void handleCommentCreateEvent(CommentCreateEvent event) {

      try {
          //send message
          kafkaTemplate.send(commentCreatedTopic, objectMapper.writeValueAsString(event));
      } catch (Exception e) {
          log.error("Got error while handling event CommentCreateEvent " + event.toString(), e);
          e.printStackTrace();
      }
  }
}
```

---

## 4. Consuming Messages

+ **메시지를 사용하고자 하는 쪽에서는 `@KafkaListener`를 사용합니다.**
+ 아래와 같은 메서드 형태로 메시지를 사용할 수 있습니다. 

```java
@KafkaListener(topics = "${YOUR_TOPIC_NAME}")
public void func(String data) {
}
```

```java
//NotificationService.java

@Service
public class NotificationService {

  //...

  //Consuming Messages
  @KafkaListener(topics = "v1.event.comment-created")
  public void subscribeCommentCreateEvent(String data) throws JsonProcessingException {

      //do something...
      //ex. parameter data 파싱
  }
}
```

---

+ 출처
    + [[스터디/9기] 단순 CRUD는 그만! 웹 백엔드 시스템 구현(Spring Boot)](https://programmers.co.kr/learn/courses/11694) 
    + [Using Kafka with Spring Boot](https://reflectoring.io/spring-boot-kafka/)
    + [Spring Boot and Kafka – Practical Example](https://thepracticaldeveloper.com/spring-boot-kafka-config/)
    + [Intro to Apache Kafka with Spring](https://www.baeldung.com/spring-kafka)

