import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.common.message.MessageExt;
import org.apache.rocketmq.common.message.Message;

public class MockConsumerWithReply {
    public static void main(String[] args) throws Exception {
        // 初始化 Consumer
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("example_consumer_group");
        consumer.setNamesrvAddr("127.0.0.1:9876");
        consumer.subscribe("RequestTopic", "*");

        consumer.registerMessageListener((MessageListenerConcurrently) (msgs, context) -> {
            for (MessageExt msg : msgs) {
                System.out.printf("Received request message: %s%n", new String(msg.getBody()));

                // 生成响应消息
                Message responseMessage = new Message("RequestTopic", "TagA", "This is a reply message".getBytes());

                try {
                    // 将响应消息通过原始消息发送回去
                    consumer.getDefaultMQPushConsumerImpl().getmQClientFactory().getProducer().reply(msg, responseMessage);
                    System.out.println("Response sent successfully.");
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
