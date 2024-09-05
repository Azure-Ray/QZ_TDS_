Here is the revised and translated guide for configuring MSK Kafka on AWS. The guide provides full URLs for the relevant MSK and Java code documentation, and the Java code is modified to use `spring-kafka` dependencies.

### Step-by-Step Guide to Configure MSK Kafka on AWS

#### 1. Create and Configure VPC and Subnets
- Create two VPCs: `routable VPC` and `non-routable VPC`.
  - The `routable VPC` is routable and can connect to on-premises via a Transit Gateway.
  - The `non-routable VPC` is not routable and contains the Network Load Balancer (NLB) and MSK.
- Create at least two subnets in each VPC (across different Availability Zones) for high availability.

#### 2. Configure VPC Endpoint (VPCe) and VPC Endpoint Service
- **VPC Endpoint (VPCe)**: Create an Interface VPC Endpoint in the `routable VPC`. It allows traffic to communicate with services in other VPCs via PrivateLink.
- **VPC Endpoint Service**: Create a VPC Endpoint Service, select the previously created Interface VPC Endpoint, and set NLB as the target.

#### 3. Configure Network Load Balancer (NLB)
- Create an NLB in the `non-routable VPC`.
  - Set up a TCP listener on the port used by MSK brokers (usually 9092).
  - Select the target group type as `IP` and add the IP addresses of the MSK brokers as targets.

#### 4. Create an MSK Cluster
- Create an Amazon MSK cluster in the `non-routable VPC`.
  - Choose a custom configuration with three broker instances.
  - In "Advanced Configuration," enable SASL/SCRAM authentication and select the option to retrieve credentials from AWS Secrets Manager. Store the username and password in Secrets Manager and ensure the secret version ID is correct.

  Documentation References:
  - Amazon MSK Cluster Creation: [https://docs.aws.amazon.com/msk/latest/developerguide/create-cluster.html](https://docs.aws.amazon.com/msk/latest/developerguide/create-cluster.html)
  - MSK Authentication using SASL/SCRAM: [https://docs.aws.amazon.com/msk/latest/developerguide/msk-authentication.html](https://docs.aws.amazon.com/msk/latest/developerguide/msk-authentication.html)

#### 5. Connect VPC Endpoint and NLB
- Configure the VPC Endpoint in the `routable VPC` to forward traffic to the NLB in the `non-routable VPC`.
- Ensure that the security group for the NLB allows traffic from the VPC Endpoint.

#### 6. Create Kafka Topic Locally Using Java with Spring-Kafka
- Ensure the Kafka client library for Spring Kafka is installed. Add the dependency to your Maven project:
  ```xml
  <dependency>
      <groupId>org.springframework.kafka</groupId>
      <artifactId>spring-kafka</artifactId>
      <version>2.8.5</version>
  </dependency>
  ```

- Write Java code using Spring Kafka to create Kafka Admin Client and create a Topic:
  ```java
  import org.apache.kafka.clients.admin.NewTopic;
  import org.springframework.kafka.core.KafkaAdmin;
  import org.springframework.kafka.core.KafkaTemplate;
  import org.springframework.kafka.core.ProducerFactory;
  import org.springframework.kafka.core.DefaultKafkaProducerFactory;
  import org.springframework.kafka.config.TopicBuilder;
  import org.springframework.kafka.core.KafkaAdmin.NewTopics;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;

  import java.util.HashMap;
  import java.util.Map;

  @Configuration
  public class KafkaTopicConfig {
      @Bean
      public KafkaAdmin kafkaAdmin() {
          Map<String, Object> config = new HashMap<>();
          config.put("bootstrap.servers", "<your-bootstrap-servers>");
          config.put("security.protocol", "SASL_SSL");
          config.put("sasl.mechanism", "SCRAM-SHA-512");
          config.put("sasl.jaas.config", "org.apache.kafka.common.security.scram.ScramLoginModule required username=\"<username>\" password=\"<password>\";");
          return new KafkaAdmin(config);
      }

      @Bean
      public NewTopic topicExample() {
          return TopicBuilder.name("my-new-topic")
                  .partitions(3)
                  .replicas(1)
                  .build();
      }
  }
  ```

#### 7. Connect to MSK Locally Using Java with Spring-Kafka
- Use SASL/SCRAM authentication to connect to MSK. Configure the Spring Kafka producer properties:
  ```java
  @Bean
  public ProducerFactory<String, String> producerFactory() {
      Map<String, Object> config = new HashMap<>();
      config.put("bootstrap.servers", "<your-bootstrap-servers>");
      config.put("security.protocol", "SASL_SSL");
      config.put("sasl.mechanism", "SCRAM-SHA-512");
      config.put("sasl.jaas.config", "org.apache.kafka.common.security.scram.ScramLoginModule required username=\"<username>\" password=\"<password>\";");
      return new DefaultKafkaProducerFactory<>(config);
  }

  @Bean
  public KafkaTemplate<String, String> kafkaTemplate() {
      return new KafkaTemplate<>(producerFactory());
  }
  ```

- When writing Kafka producer or consumer code, pass the above configurations to the Kafka Producer or Consumer instance.

  Documentation References:
  - Kafka Clients Documentation: [https://kafka.apache.org/documentation/#producerapi](https://kafka.apache.org/documentation/#producerapi)
  - Secure Kafka Java Client with Spring Kafka: [https://docs.spring.io/spring-kafka/docs/current/reference/html/#_sasl_ssl](https://docs.spring.io/spring-kafka/docs/current/reference/html/#_sasl_ssl)

### Conclusion
By following the steps above, you will successfully configure an MSK Kafka cluster on AWS and be able to connect and manage the Kafka cluster locally using Java with Spring-Kafka. Be sure to consult the official documentation to ensure your setup meets your specific network and security requirements.
