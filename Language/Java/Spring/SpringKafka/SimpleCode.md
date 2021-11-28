```properties
server.port=8080
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=app.1
```

```java
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.IntegerSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;
import org.springframework.kafka.support.serializer.JsonSerializer;
import ru.job4j.mail.dto.UserDTO;

import java.util.HashMap;
import java.util.Map;

@Configuration
public class KafkaProducerConfig {

    @Value("${spring.kafka.bootstrap-servers}")
    private String kafkaServer;

    @Bean
    public KafkaTemplate<Integer, UserDTO> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }

    @Bean
    public ProducerFactory<Integer, UserDTO> producerFactory() {
        return new DefaultKafkaProducerFactory<>(producerConfigs());
    }

    @Bean
    public Map<String, Object> producerConfigs() {
        Map<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, kafkaServer);
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, IntegerSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
        return props;
    }

}
```

```java
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.IntegerDeserializer;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.config.KafkaListenerContainerFactory;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import ru.job4j.mail.dto.UserDTO;

import java.util.HashMap;
import java.util.Map;

@Configuration
public class KafkaConsumerConfig {

    @Value("${spring.kafka.bootstrap-servers}")
    private String kafkaServer;

    @Value("${spring.kafka.consumer.group-id}")
    private String kafkaGroupId;

    @Bean
    public Map<String, Object> consumerConfigs() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, kafkaServer);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, IntegerDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.GROUP_ID_CONFIG, kafkaGroupId);
        return props;
    }

    @Bean
    public KafkaListenerContainerFactory<?> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<Long, UserDTO> factory =
                new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }

    @Bean
    public ConsumerFactory<Long, UserDTO> consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(consumerConfigs());
    }
}
```

```java
@EnableKafka
@RestController
@RequestMapping("/console")
public class ConsoleController {

    @KafkaListener(topics="msg")
    public void msgListener(ConsumerRecord<Integer, UserDTO> message){
        System.out.println("Message: topic = " + message.topic());
        System.out.println("Message: key = " + message.key());
        System.out.println("Message: value = " + message.value());
    }

}
```

```java
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.SendResult;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.util.concurrent.ListenableFuture;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import ru.job4j.mail.dto.AddressDTO;
import ru.job4j.mail.dto.UserDTO;

import java.util.List;
import java.util.Random;

@RestController
@RequestMapping("/msg")
public class MailController {
    private final KafkaTemplate<Integer, UserDTO> kafkaTemplate;
    private final List<String> randomValues;

    public MailController(KafkaTemplate<Integer, UserDTO> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
        randomValues = List.of("Hello", "End", "Test");
    }

    @PostMapping
    public void send(Integer key, UserDTO value) {
        System.out.println("key: " + key);
        System.out.println("value: " + value);
        ListenableFuture<SendResult<Integer, UserDTO>> future = kafkaTemplate.send("msg", key, value);
        future.addCallback(System.out::println, System.err::println);
        kafkaTemplate.flush();
    }

    @Scheduled(fixedRate = 3000)
    public void rateSend() {
        int rnd = new Random().nextInt(3);
        var user = new UserDTO();
        user.setName("Dima");
        user.setAge(31L);
        user.setAddress(new AddressDTO("Russia", "Ekaterinburg", 308L));
        send(rnd, user);
    }
}
```