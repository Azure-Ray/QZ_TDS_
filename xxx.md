import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.RequestResponseFuture;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;

import java.util.concurrent.TimeUnit;

public class SyncProducerWithReply {
    public static void main(String[] args) throws Exception {
        // 初始化Producer
        DefaultMQProducer producer = new DefaultMQProducer("example_producer_group");
        producer.setNamesrvAddr("127.0.0.1:9876");
        producer.start();

        // 创建消息
        Message msg = new Message("RequestTopic", "TagA", "Hello RocketMQ".getBytes());

        // 发送消息并等待响应
        RequestResponseFuture responseFuture = producer.request(msg, 3000);

        // 接收并处理响应
        Message replyMessage = responseFuture.get(3000, TimeUnit.MILLISECONDS);
        if (replyMessage != null) {
            System.out.printf("Received reply message: %s%n", new String(replyMessage.getBody()));
        } else {
            System.out.println("No reply message received within the timeout period.");
        }

        producer.shutdown();
    }
}

import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.common.message.MessageExt;

public class MockConsumerWithReply {
    public static void main(String[] args) throws Exception {
        // 配置消费消息的 Consumer
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("example_consumer_group");
        consumer.setNamesrvAddr("127.0.0.1:9876");
        consumer.subscribe("RequestTopic", "*");

        // 启动Consumer
        consumer.registerMessageListener((MessageListenerConcurrently) (msgs, context) -> {
            for (MessageExt msg : msgs) {
                System.out.printf("Received request message: %s%n", new String(msg.getBody()));

                // 创建响应消息
                Message responseMessage = new Message("RequestTopic", "TagA", "This is a reply message".getBytes());

                // 发送响应消息
                try {
                    // 通过 RequestId 来匹配请求和响应
                    responseMessage.setProperty("CORRELATION_ID", msg.getProperty("CORRELATION_ID"));
                    DefaultMQProducer producer = new DefaultMQProducer("response_producer_group");
                    producer.setNamesrvAddr("127.0.0.1:9876");
                    producer.start();
                    SendResult sendResult = producer.send(responseMessage);
                    System.out.printf("Response sent: %s%n", sendResult);
                    producer.shutdown();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        });

        consumer.start();
        System.out.printf("MockConsumer Started.%n");
    }
}

