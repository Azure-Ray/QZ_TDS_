package com.example.otherproject.controller;

import com.example.kafka.KafkaProducerService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/messages")
public class MessageController {

    private final KafkaProducerService kafkaProducerService;

    @Autowired
    public MessageController(KafkaProducerService kafkaProducerService) {
        this.kafkaProducerService = kafkaProducerService;
    }

    @PostMapping("/send")
    public String sendMessage(@RequestParam String message) {
        kafkaProducerService.sendMessage(message);
        return "Message sent: " + message;
    }
}
