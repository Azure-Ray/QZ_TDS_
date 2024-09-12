下面是基于Spring Boot的项目结构，提供REST API来启动Kafka的Producer和Consumer。这将使项目更易于测试，同时仍然可以被其他项目作为依赖使用。

### 1. 更新 `pom.xml`

首先，`pom.xml`需要进行一些更新以支持Spring Boot项目和REST API功能：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example.kafka</groupId>
    <artifactId>kafka-api</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <java.version>1.8</java.version>
        <spring.kafka.version>2.7.12</spring.kafka.version>
        <kafka.clients.version>2.8.2</kafka.clients.version>
    </properties>

    <dependencies>
        <!-- Spring Boot Starter Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.5.12</version>
        </dependency>

        <!-- Spring Kafka -->
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
            <version>${spring.kafka.version}</version>
        </dependency>

        <!-- Kafka Clients -->
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>${kafka.clients.version}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Maven Compiler Plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>

            <!-- Spring Boot Maven Plugin -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### 2. 配置文件 (`application.properties`)

继续使用配置文件来管理Kafka的连接和安全设置。创建或更新`src/main/resources/application.properties`文件：

```properties
# Kafka Producer 配置
spring.kafka.bootstrap-servers=broker1:9093,broker2:9093,broker3:9093
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer

# Kafka Consumer 配置
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.group-id=my-group

# SASL_SSL 安全配置
spring.kafka.properties.security.protocol=SASL_SSL
spring.kafka.properties.sasl.mechanism=PLAIN
spring.kafka.properties.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="your-username" password="your-password";
```

### 3. 生产者和消费者服务类

创建Kafka生产者和消费者服务类，它们将提供API来发送和接收消息。

#### 生产者服务 (`KafkaProducerService.java`)

```java
package com.example.kafka;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;

@Service
public class KafkaProducerService {

    private final KafkaTemplate<String, String> kafkaTemplate;

    @Value("${spring.kafka.producer.topic:default-topic}")
    private String topic;

    public KafkaProducerService(KafkaTemplate<String, String> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void sendMessage(String message) {
        kafkaTemplate.send(topic, message);
        System.out.println("Sent message: " + message);
    }
}
```

#### 消费者服务 (`KafkaConsumerService.java`)

```java
package com.example.kafka;

import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

@Service
public class KafkaConsumerService {

    @KafkaListener(topics = "${spring.kafka.consumer.topic:default-topic}", groupId = "${spring.kafka.consumer.group-id}")
    public void listen(String message) {
        System.out.println("Received message: " + message);
    }
}
```

### 4. REST 控制器 (`KafkaController.java`)

创建一个REST控制器来提供API端点，允许用户通过API启动Kafka Producer和Consumer。

```java
package com.example.kafka;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/kafka")
public class KafkaController {

    private final KafkaProducerService producerService;

    @Autowired
    public KafkaController(KafkaProducerService producerService) {
        this.producerService = producerService;
    }

    @PostMapping("/send")
    public String sendMessage(@RequestParam("message") String message) {
        producerService.sendMessage(message);
        return "Message sent successfully!";
    }

    @GetMapping("/receive")
    public String receiveMessage() {
        // 实际上Consumer是自动监听的，这里仅用作示例
        return "Messages are being consumed automatically!";
    }
}
```

### 5. 主应用程序类 (`KafkaApplication.java`)

编写主应用程序类来启动Spring Boot应用：

```java
package com.example.kafka;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class KafkaApplication {

    public static void main(String[] args) {
        SpringApplication.run(KafkaApplication.class, args);
    }
}
```

### 6. 打包和运行

使用以下命令来打包项目并生成JAR文件：

```bash
mvn clean package
```

生成的JAR文件会位于`target`目录中，可以通过以下命令运行：

```bash
java -jar target/kafka-api-1.0-SNAPSHOT.jar
```

### 7. 测试API

启动Spring Boot应用后，可以通过以下API端点测试Producer和Consumer：

- **发送消息**（POST请求）：
  ```bash
  curl -X POST "http://localhost:8080/api/kafka/send?message=HelloKafka"
  ```
- **接收消息**（GET请求）：
  ```bash
  curl -X GET "http://localhost:8080/api/kafka/receive"
  ```

### 8. 用作依赖

将这个项目打包成JAR文件后，可以在其他项目的`pom.xml`中添加这个JAR文件作为依赖：

```xml
<dependency>
    <groupId>com.example.kafka</groupId>
    <artifactId>kafka-api</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

这样，您就可以在其他项目中复用此Kafka Producer和Consumer的功能，同时也可以通过API端点进行测试。
