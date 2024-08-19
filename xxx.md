package com.example.rocketmqmockservice;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import java.io.File;
import java.io.IOException;

@SpringBootApplication
public class RocketMQMockServiceApplication implements CommandLineRunner {

    public static void main(String[] args) {
        SpringApplication.run(RocketMQMockServiceApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        startRocketMQ();
    }

    private void startRocketMQ() {
        try {
            // 启动 NameServer
            ProcessBuilder nameSrvPb = new ProcessBuilder(
                "cmd.exe", "/c", "start", "mqnamesrv.cmd");
            nameSrvPb.directory(new File("src/main/resources/rocketmq/bin"));
            nameSrvPb.inheritIO();
            Process nameSrvProcess = nameSrvPb.start();
            
            // 启动 Broker
            ProcessBuilder brokerPb = new ProcessBuilder(
                "cmd.exe", "/c", "start", "mqbroker.cmd", "-n", "127.0.0.1:9876", "autoCreateTopicEnable=true");
            brokerPb.directory(new File("src/main/resources/rocketmq/bin"));
            brokerPb.inheritIO();
            Process brokerProcess = brokerPb.start();

        } catch (IOException e) {
            e.printStackTrace();
            throw new RuntimeException("Failed to start RocketMQ", e);
        }
    }
}



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
