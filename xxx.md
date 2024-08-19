rocketmq:
  name-server: 127.0.0.1:9876
  producer:
    group: test_producer_group
    enabled: true
  consumer:
    group: test_consumer_group
    enabled: true
    topic: RequestTopic


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





#!/bin/bash

# 启动 NameServer
nohup sh ./bin/mqnamesrv &

# 启动 Broker
nohup sh ./bin/mqbroker -n 127.0.0.1:9876 autoCreateTopicEnable=true &


package com.example.rocketmqmockservice;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

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
            ProcessBuilder pb = new ProcessBuilder("sh", "./src/main/resources/rocketmq/start-rocketmq.sh");
            pb.inheritIO();
            Process process = pb.start();
            int exitCode = process.waitFor();
            if (exitCode != 0) {
                throw new RuntimeException("Failed to start RocketMQ");
            }
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
        }
    }
}

package com.example.mockmq.service;

import org.springframework.stereotype.Service;

import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

@Service
public class MockMQService {

    private final BlockingQueue<String> queue = new LinkedBlockingQueue<>();

    // 生产消息（模拟发送消息）
    public void produce(String message) {
        try {
            queue.put(message);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    // 消费消息（模拟接收消息）
    public String consume() {
        try {
            return queue.take();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return null;
        }
    }
}

package com.example.mockmq.controller;

import com.example.mockmq.service.MockMQService;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/mq")
public class MockMQController {

    private final MockMQService mockMQService;

    public MockMQController(MockMQService mockMQService) {
        this.mockMQService = mockMQService;
    }

    // 生产消息的API
    @PostMapping("/produce")
    public String produceMessage(@RequestParam String message) {
        mockMQService.produce(message);
        return "Message produced: " + message;
    }

    // 消费消息的API
    @GetMapping("/consume")
    public String consumeMessage() {
        String message = mockMQService.consume();
        return message != null ? "Message consumed: " + message : "No messages available";
    }
}


curl -X POST "http://localhost:8080/mq/produce?message=Hello+World"


curl -X GET "http://localhost:8080/mq/consume"
