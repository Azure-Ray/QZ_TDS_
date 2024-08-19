package com.example.rocketmqmockservice;

import org.apache.rocketmq.spring.annotation.RocketMQMessageListener;
import org.apache.rocketmq.spring.core.RocketMQTemplate;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.messaging.Message;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.stereotype.Service;

@SpringBootApplication
public class RocketMQMockServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(RocketMQMockServiceApplication.class, args);
    }
}

@Service
class MessageProcessor {

    private final RocketMQTemplate rocketMQTemplate;

    public MessageProcessor(RocketMQTemplate rocketMQTemplate) {
        this.rocketMQTemplate = rocketMQTemplate;
    }

    // 消息监听器，用于接收消息并处理
    @RocketMQMessageListener(topic = "RequestTopic", consumerGroup = "test_consumer_group")
    public void onMessage(String receivedMessage) {
        System.out.println("Received message: " + receivedMessage);

        // 根据接收到的消息判断响应内容
        String responseContent = "Fixed Response";
        if ("special request".equalsIgnoreCase(receivedMessage)) {
            responseContent = "Special Response";
        }

        // 发送响应消息到 ResponseTopic
        Message<String> message = MessageBuilder.withPayload(responseContent).build();
        rocketMQTemplate.convertAndSend("ResponseTopic", message);
    }
}
