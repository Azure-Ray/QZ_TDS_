
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.apache.kafka.common.serialization.StringSerializer;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.test.context.EmbeddedKafka;
import org.springframework.kafka.test.utils.KafkaTestUtils;
import org.springframework.kafka.test.EmbeddedKafkaBroker;
import org.springframework.kafka.test.utils.ContainerTestUtils;
import org.springframework.kafka.listener.MessageListenerContainer;
import org.springframework.kafka.listener.config.ContainerProperties;
import org.springframework.kafka.test.context.EmbeddedKafka;

import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.TimeUnit;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest
@EmbeddedKafka(partitions = 1, topics = { "test-topic" })
public class KafkaMqClientProviderTest {

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    @Autowired
    private EmbeddedKafkaBroker embeddedKafka;

    private BlockingQueue<ConsumerRecord<String, String>> records;

    @BeforeEach
    public void setUp() {
        records = new LinkedBlockingQueue<>();

        // Set up a consumer for testing
        ContainerProperties containerProps = new ContainerProperties("test-topic");
        containerProps.setGroupId("test-group");
        containerProps.setMessageListener((org.springframework.kafka.listener.MessageListener<String, String>) record -> records.add(record));

        MessageListenerContainer container = new ConcurrentMessageListenerContainer<>(
                new DefaultKafkaConsumerFactory<>(
                        KafkaTestUtils.consumerProps("test-group", "true", embeddedKafka),
                        new StringDeserializer(),
                        new StringDeserializer()
                ),
                containerProps
        );

        container.start();
        ContainerTestUtils.waitForAssignment(container, embeddedKafka.getPartitionsPerTopic());
    }

    @Test
    public void testSendMessage() throws InterruptedException {
        // Given
        String message = "Hello, Kafka!";
        String topic = "test-topic";

        // When
        kafkaTemplate.send(topic, message);

        // Then
        ConsumerRecord<String, String> received = records.poll(10, TimeUnit.SECONDS);
        assertThat(received).isNotNull();
        assertThat(received.value()).isEqualTo(message);
    }
}
