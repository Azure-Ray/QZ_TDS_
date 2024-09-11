
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.apache.kafka.common.serialization.StringSerializer;
import org.jetbrains.annotations.NotNull;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;
import org.springframework.kafka.listener.ConcurrentMessageListenerContainer;
import org.springframework.kafka.listener.MessageListenerContainer;
import org.springframework.kafka.listener.config.ContainerProperties;
import org.springframework.kafka.support.serializer.ErrorHandlingDeserializer;
import org.springframework.kafka.support.serializer.ErrorHandlingDeserializer2;
import org.springframework.kafka.support.serializer.StringDeserializer;
import org.springframework.kafka.support.serializer.StringSerializer;

import java.util.HashMap;
import java.util.Map;

@Configuration
public class KafkaMqClientProvider {

    @Value("${kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Value("${kafka.topic}")
    private String topic;

    @Value("${wng.stub.delay:500}")
    private int responseDelayTime;

    private Map<String, String> stubMap = new HashMap<>();

    @Bean
    public ProducerFactory<String, String> producerFactory() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        return new DefaultKafkaProducerFactory<>(configProps);
    }

    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }

    @Bean
    public ConsumerFactory<String, String> consumerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "your-group-id");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        return new DefaultKafkaConsumerFactory<>(props);
    }

    @Bean
    public MessageListenerContainer messageListenerContainer() {
        ContainerProperties containerProps = new ContainerProperties(topic);
        containerProps.setMessageListener(new KafkaMessageListener());
        return new ConcurrentMessageListenerContainer<>(consumerFactory(), containerProps);
    }

    public void sendMessage(String message) {
        KafkaTemplate<String, String> kafkaTemplate = kafkaTemplate();
        kafkaTemplate.send(topic, message);
        if (responseDelayTime > 0) {
            try {
                Thread.sleep(responseDelayTime);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                throw new RuntimeException(e);
            }
        }
    }

    @KafkaListener(topics = "${kafka.topic}", groupId = "your-group-id")
    public void receiveMessage(String message) {
        // Processing incoming Kafka messages
        if (!stubMap.containsKey(message)) {
            System.err.println("Stub request: " + message);
            throw new RuntimeException("Stub data not found!");
        } else {
            String response = stubMap.get(message);
            System.out.println("Received response: " + response);
        }
    }

    private void loadStubData(String prefix) {
        stubMap.put(getDataString("classpath:stub/" + prefix + "_request.txt"),
                getDataString("classpath:stub/" + prefix + "_response.txt"));
    }

    private String getDataString(String resourcePath) {
        try {
            return IOUtils.toString(ResourceUtils.getURI(resourcePath), StandardCharsets.UTF_8);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    static {
        System.out.println("Stub Kafka Client Enabled!");
    }
}

class KafkaMessageListener implements org.springframework.kafka.listener.MessageListener<String, String> {

    @Override
    public void onMessage(org.apache.kafka.clients.consumer.ConsumerRecord<String, String> record) {
        System.out.println("Received Kafka message: " + record.value());
        // Process the message
    }
}
