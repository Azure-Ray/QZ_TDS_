下面是一套基于JDK 8的简单Kafka Producer和Consumer代码示例，它可以打包成一个JAR文件供他人使用。Kafka的连接参数和安全配置都通过配置文件来控制，使其更加灵活。

### 1. Maven 配置 (`pom.xml`)

首先，配置Maven的`pom.xml`文件，指定Kafka和Spring Kafka的版本：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example.kafka</groupId>
    <artifactId>kafka-producer-consumer</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <java.version>1.8</java.version>
        <spring.kafka.version>2.7.12</spring.kafka.version>
        <kafka.clients.version>2.8.2</kafka.clients.version>
    </properties>

    <dependencies>
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

        <!-- Spring Boot Starter -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <version>2.5.12</version>
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

创建一个`src/main/resources/application.properties`文件，用于配置Kafka的参数。

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

### 3. 生产者代码 (`KafkaProducerService.java`)

编写一个简单的Kafka Producer类，用于发送消息：

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

### 4. 消费者代码 (`KafkaConsumerService.java`)

编写一个简单的Kafka Consumer类，用于接收消息：

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

### 5. 主应用程序类 (`KafkaApplication.java`)

编写主应用程序类来启动Spring Boot应用：

```java
package com.example.kafka;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class KafkaApplication {

    public static void main(String[] args) {
        SpringApplication.run(KafkaApplication.class, args);
    }

    @Bean
    public CommandLineRunner commandLineRunner(KafkaProducerService producerService) {
        return args -> {
            // 发送测试消息
            producerService.sendMessage("Hello, Kafka!");
        };
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
java -jar target/kafka-producer-consumer-1.0-SNAPSHOT.jar
```

### 7. 配置示例

确保在运行时提供正确的Kafka配置和凭据，特别是`sasl.jaas.config`部分需要根据实际环境进行修改。

这套代码和配置应该能够满足你的要求，并允许其他人直接调用，配置Kafka参数时更加灵活。
