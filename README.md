import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.Acknowledgment;
import org.springframework.kafka.support.KafkaHeaders;
import org.springframework.messaging.Message;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.stereotype.Service;

import java.util.Map;
import java.util.UUID;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.TimeUnit;

@Service
public class KafkaConsumerService {

    private final KafkaTemplate<String, String> kafkaTemplate;

    // 用于存储每个请求的响应队列
    private final Map<String, BlockingQueue<String>> responseMap = new ConcurrentHashMap<>();

    @Value("${spring.kafka.producer.topic:default-topic}")
    private String requestTopic;

    private static final String TOPIC_BILLER_QRY_RESPONSE = "msk_response_a_test_3";

    public KafkaConsumerService(KafkaTemplate<String, String> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    @KafkaListener(id = "response-listener", topics = TOPIC_BILLER_QRY_RESPONSE,
            groupId = "${spring.kafka.consumer.group-id:msk_request_9}",
            containerFactory = "kafkaListenerContainerFactory")
    public void readMessage(String message, @Header(KafkaHeaders.CORRELATION_ID) String correlationId, Acknowledgment acknowledgment) {
        acknowledgment.acknowledge();
        System.out.println("Received message: " + message + " for Correlation ID: " + correlationId);

        // 查找对应的请求的 BlockingQueue，并将消息放入队列
        BlockingQueue<String> queue = responseMap.get(correlationId);
        if (queue != null) {
            queue.offer(message);  // 将响应消息放入队列中
        }
    }

    public String sendAndReceive(String message) throws InterruptedException {
        String correlationId = createCorrelationId();

        // 为每个请求创建一个阻塞队列来等待响应
        BlockingQueue<String> responseQueue = new LinkedBlockingQueue<>();
        responseMap.put(correlationId, responseQueue);

        Message<String> messageToSend = MessageBuilder.withPayload(message)
                .setHeader(KafkaHeaders.TOPIC, requestTopic)
                .setHeader(KafkaHeaders.CORRELATION_ID, correlationId)
                .build();

        // 同步发送消息
        kafkaTemplate.send(messageToSend);

        try {
            // 等待最多 30 秒，直到有响应进入队列
            String response = responseQueue.poll(30, TimeUnit.SECONDS);
            if (response == null) {
                throw new RuntimeException("Timeout waiting for response with correlationId: " + correlationId);
            }
            return response;
        } finally {
            // 请求完成后，清理对应的阻塞队列
            responseMap.remove(correlationId);
        }
    }

    public String createCorrelationId() {
        return UUID.randomUUID().toString();
    }
}
