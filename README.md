### Guide: Performance Testing POC with MSK Kafka as Mock MQ

This guide covers the performance testing proof of concept (POC) where **MSK Kafka** is utilized as a mock message queue (MQ). The POC is divided into three parts:
1. **Dependencies and Configuration for Project**: The project uses **JDK 8** with **Spring Boot 2.7**, and the dependencies involve `spring-kafka` and `kafka-client` to connect to MSK Kafka.
2. **Mock MQ Server (Downstream System)**: This part uses **JDK 17**, **Spring Boot 2.7**, and `spring-kafka` to simulate the MQ server.
3. **MSK Kafka**: Version **2.8.2** is used as the core messaging platform.

In this guide, we will focus on the first two parts, detailing the dependencies, configurations, and how to set up the mock server.

---

## Part 1: Project Dependencies and Usage

The project’s code utilizes **MSK Kafka** for handling messaging via `sendAndReceive` methods. This section explains how to configure and integrate the necessary dependencies for the project.

### Maven Dependencies

In your **pom.xml**, include the following dependencies to connect to MSK Kafka using **Spring Boot 2.7** and **JDK 8**:

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>2.7.9</version>
</dependency>
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.8.2</version>
</dependency>
```

These dependencies provide the necessary components for interacting with MSK Kafka.

### Code Example

In your code, you can send and receive messages using the `sendAndReceive` method. The method accepts the **request** and returns a **response** (as a `String`). Here's a simplified example:

```java
@Autowired
private KafkaTemplate<String, String> kafkaTemplate;

public String sendAndReceive(String request) {
    return kafkaTemplate.sendAndReceive("yourTopic", request).get().value();
}
```

This code sends a message to the Kafka topic and waits for a response. The `String` response is the return value.

#### How the Code Works

- **Correlation ID**: The Kafka producer attaches a unique `correlationId` to the message header. This ID is used to track the request and match it with the corresponding response.
- **Asynchronous Processing**: The message is sent asynchronously, and the listener listens for the response message with the same `correlationId`.
- **Response Handling**: Once the message is received, the response is returned within the same method.

### Configuration

In **application.yml** (or **application.properties**), configure the Kafka broker settings to connect to **MSK Kafka**:

```yaml
spring:
  kafka:
    bootstrap-servers: <MSK-Broker-Endpoint>
    consumer:
      group-id: your-group-id
      auto-offset-reset: earliest
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    listener:
      ack-mode: manual
```

For further configuration options, you can refer to the official Kafka documentation here:  
https://kafka.apache.org/28/documentation.html

---

## Part 2: Mock MQ Server

The second part of the POC is setting up the **Mock MQ Server**. This server simulates downstream systems and processes **request** and **response** messages. The server uses **JDK 17**, **Spring Boot 2.7**, and `spring-kafka`.

### Configuration

Like the project dependencies, the mock server also connects to MSK Kafka. Here’s an outline of the configuration and functionality:

1. **Maven Dependencies**: Same as the ones mentioned in Part 1, with additional Spring Boot dependencies for configuring the server.
   
2. **Message Processing**: The mock server processes incoming requests by listening on Kafka topics, and based on the message type, it returns a pre-configured response. 

### Storing Mock Data in Git

The mock server allows dynamic configuration of **request** and **response** pairs stored in Git repositories. The mock data can be modified online, enabling or disabling specific mock responses without restarting the server.

The request-response configurations can be set up as simple JSON files in a Git repository. For example:

```json
{
  "request": "sampleRequest",
  "response": "sampleResponse"
}
```

These files are fetched by the server on-demand, so any changes in the Git repository will be reflected in the mock behavior without requiring a redeployment.

To configure this, add the following properties in **application.yml**:

```yaml
mock:
  data:
    git-repository: https://your-git-repo-url
    auto-reload: true
```

This will enable the mock server to fetch request-response mappings from Git and auto-reload the configurations dynamically.

### Code Example

Here’s a simple example of how the mock server listens for incoming requests and responds with pre-configured messages:

```java
@KafkaListener(topics = "yourTopic", groupId = "mock-group")
public void processRequest(ConsumerRecord<String, String> record) {
    String request = record.value();
    String response = fetchMockResponse(request);
    kafkaTemplate.send("yourResponseTopic", response);
}
```

The method `fetchMockResponse` would query the Git repository for the pre-configured response corresponding to the incoming request.

For more information on setting up Kafka listeners and producers, refer to the official Spring Kafka documentation:  
https://docs.spring.io/spring-kafka/docs/current/reference/html/

---

## Part 3: MSK Kafka Setup

The third component is **MSK Kafka** itself. This POC uses **Kafka 2.8.2**, hosted on AWS MSK.

To set up **MSK Kafka**, refer to the official AWS documentation here:  
https://docs.aws.amazon.com/msk/latest/developerguide/what-is-msk.html

### Steps to Configure MSK Kafka:

1. **Create an MSK Cluster**: Follow the AWS guide to create an MSK Kafka cluster.
2. **Configure Security**: Ensure proper security group settings and authentication using the MSK configuration.
3. **Connect from Spring Kafka**: Use the MSK broker endpoint in your **application.yml** configuration, as mentioned in Part 1.

---

This guide should give you a clear starting point to set up the **MSK Kafka POC** for performance testing. For more detailed implementation and advanced configurations, refer to the following additional resources:

- **Kafka Documentation**: https://kafka.apache.org/documentation
- **Spring Kafka Documentation**: https://docs.spring.io/spring-kafka/docs/current/reference/html/
- **AWS MSK**: https://docs.aws.amazon.com/msk/latest/developerguide/what-is-msk.html
