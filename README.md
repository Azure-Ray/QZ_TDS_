将上述 RocketMQ 服务从 Windows 搬到 Linux 环境，并实现相同的功能，下面是一个完整的流程，包括安装、配置、编写和运行 Java 文件，以及打包成 JAR 文件。

环境准备
安装 Java：
确保你的 Linux 环境已经安装了 Java Development Kit (JDK)。

bash
复制代码
sudo apt update
sudo apt install openjdk-11-jdk -y
下载并安装 RocketMQ：

bash
复制代码
wget https://archive.apache.org/dist/rocketmq/4.9.2/rocketmq-all-4.9.2-bin-release.zip
unzip rocketmq-all-4.9.2-bin-release.zip -d /opt/rocketmq
cd /opt/rocketmq
设置环境变量：
编辑 ~/.bashrc 或 ~/.profile 文件，添加以下内容：

bash
复制代码
export ROCKETMQ_HOME=/opt/rocketmq
export PATH=$PATH:$ROCKETMQ_HOME/bin
然后执行 source ~/.bashrc 或 source ~/.profile 使环境变量生效。

启动 RocketMQ 服务
启动 NameServer：

bash
复制代码
nohup sh $ROCKETMQ_HOME/bin/mqnamesrv &
启动 Broker：

bash
复制代码
nohup sh $ROCKETMQ_HOME/bin/mqbroker -n 127.0.0.1:9876 autoCreateTopicEnable=true &
编写和打包 Java 文件
创建项目结构：

bash
复制代码
mkdir -p ~/rocketmq-mock/src/main/java/com/example/rocketmq
cd ~/rocketmq-mock
编写 pom.xml 文件：
在项目根目录下创建 pom.xml 文件：

xml
复制代码
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>rocketmq-mock</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.apache.rocketmq</groupId>
            <artifactId>rocketmq-client</artifactId>
            <version>4.9.2</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.3.0</version>
                <configuration>
                    <archive>
                        <manifest>
                            <mainClass>com.example.rocketmq.MockConsumer</mainClass>
                        </manifest>
                    </archive>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
编写 Java 文件：

Producer.java：

java
复制代码
package com.example.rocketmq;

import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;

public class Producer {
    public static void main(String[] args) throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("test_producer_group");
        producer.setNamesrvAddr("127.0.0.1:9876");
        producer.start();

        for (int i = 0; i < 10; i++) {
            Message msg = new Message("RequestTopic", "TagA", ("Hello RocketMQ " + i).getBytes());
            SendResult sendResult = producer.send(msg);
            System.out.printf("Message sent: %s%n", sendResult);
        }
        producer.shutdown();
    }
}
MockConsumer.java：

java
复制代码
package com.example.rocketmq;

import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.common.message.MessageExt;

public class MockConsumer {
    public static void main(String[] args) throws Exception {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("test_consumer_group");
        consumer.setNamesrvAddr("127.0.0.1:9876");
        consumer.subscribe("RequestTopic", "*");

        DefaultMQProducer responseProducer = new DefaultMQProducer("response_producer_group");
        responseProducer.setNamesrvAddr("127.0.0.1:9876");
        responseProducer.start();

        consumer.registerMessageListener((MessageListenerConcurrently) (msgs, context) -> {
            for (MessageExt msg : msgs) {
                System.out.printf("Received message: %s %n", new String(msg.getBody()));
                Message responseMsg = new Message("ResponseTopic", "TagA", "Fixed Response".getBytes());
                try {
                    SendResult sendResult = responseProducer.send(responseMsg);
                    System.out.printf("Response sent: %s%n", sendResult);
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
ResponseConsumer.java：

java
复制代码
package com.example.rocketmq;

import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.common.message.MessageExt;

public class ResponseConsumer {
    public static void main(String[] args) throws Exception {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("response_consumer_group");
        consumer.setNamesrvAddr("127.0.0.1:9876");
        consumer.subscribe("ResponseTopic", "*");

        consumer.registerMessageListener((MessageListenerConcurrently) (msgs, context) -> {
            for (MessageExt msg : msgs) {
                System.out.printf("Received response message: %s %n", new String(msg.getBody()));
            }
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        });

        consumer.start();
        System.out.printf("ResponseConsumer Started.%n");
    }
}
编译并打包项目：
在项目根目录下运行以下命令，编译并打包项目：

bash
复制代码
mvn clean compile assembly:single
这将生成一个带有所有依赖项的 JAR 文件，位于 target 目录下，名为 rocketmq-mock-1.0-SNAPSHOT-jar-with-dependencies.jar。

运行程序
启动 MockConsumer：

bash
复制代码
java -cp target/rocketmq-mock-1.0-SNAPSHOT-jar-with-dependencies.jar com.example.rocketmq.MockConsumer
运行 Producer 发送测试消息：

bash
复制代码
java -cp target/rocketmq-mock-1.0-SNAPSHOT-jar-with-dependencies.jar com.example.rocketmq.Producer
启动 ResponseConsumer 接收响应：

bash
复制代码
java -cp target/rocketmq-mock-1.0-SNAPSHOT-jar-with-dependencies.jar com.example.rocketmq.ResponseConsumer
解释
MockConsumer：接收 RequestTopic 的消息并发送固定响应消息到 ResponseTopic。
ResponseConsumer：接收 ResponseTopic 的响应消息并打印出来。
通过这些步骤，你可以在 Linux 环境下实现 RocketMQ 的 Mock 服务，并返回固定的响应内容。

然后别人怎么call我这个mq呢？
ChatGPT
要使其他人能够调用你的 RocketMQ 服务并接收响应消息，可以提供一个清晰的接口和示例代码，指导他们如何发送消息到你的 RocketMQ 实例并接收响应消息。

步骤概述
提供 RocketMQ 配置：告诉他们 NameServer 地址和 Topic 信息。
示例代码：提供一个 Producer 示例代码，展示如何发送消息到你的 RocketMQ 服务。
响应处理：说明如何订阅并接收响应消息。
1. 提供 RocketMQ 配置
告知他们你的 RocketMQ NameServer 地址和使用的 Topic。假设你的 NameServer 地址是 127.0.0.1:9876，发送消息的 Topic 是 RequestTopic，接收响应的 Topic 是 ResponseTopic。

2. 示例代码
提供一个示例代码，展示如何发送消息到你的 RocketMQ 服务并接收响应消息。

Producer 示例代码：

java
复制代码
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;

public class MQProducer {
    public static void main(String[] args) throws Exception {
        // 初始化Producer
        DefaultMQProducer producer = new DefaultMQProducer("example_producer_group");
        producer.setNamesrvAddr("127.0.0.1:9876");
        producer.start();

        // 发送消息
        Message msg = new Message("RequestTopic", "TagA", "Test message".getBytes());
        SendResult sendResult = producer.send(msg);
        System.out.printf("Message sent: %s%n", sendResult);

        // 关闭Producer
        producer.shutdown();
    }
}
Consumer 示例代码：

java
复制代码
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.common.message.MessageExt;

public class MQConsumer {
    public static void main(String[] args) throws Exception {
        // 初始化Consumer
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("example_consumer_group");
        consumer.setNamesrvAddr("127.0.0.1:9876");
        consumer.subscribe("ResponseTopic", "*");

        // 注册消息监听器
        consumer.registerMessageListener((MessageListenerConcurrently) (msgs, context) -> {
            for (MessageExt msg : msgs) {
                System.out.printf("Received response message: %s %n", new String(msg.getBody()));
            }
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        });

        // 启动Consumer
        consumer.start();
        System.out.printf("Consumer Started.%n");
    }
}
3. 如何使用
将以下步骤提供给其他用户，让他们了解如何调用你的 RocketMQ 服务并接收响应消息。

步骤
确保依赖项：在他们的项目中添加 RocketMQ 的 Maven 依赖项。
编写并运行 Producer：使用提供的 Producer 示例代码发送消息。
编写并运行 Consumer：使用提供的 Consumer 示例代码接收响应消息。
Maven 依赖项
确保他们在 pom.xml 文件中添加以下依赖项：

xml
复制代码
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-client</artifactId>
    <version>4.9.2</version>
</dependency>
编写并运行 Producer
让他们编写并运行 Producer 代码，将消息发送到你的 RocketMQ 服务：

java
复制代码
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;

public class MQProducer {
    public static void main(String[] args) throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("example_producer_group");
        producer.setNamesrvAddr("127.0.0.1:9876");
        producer.start();

        Message msg = new Message("RequestTopic", "TagA", "Test message".getBytes());
        SendResult sendResult = producer.send(msg);
        System.out.printf("Message sent: %s%n", sendResult);

        producer.shutdown();
    }
}
编写并运行 Consumer
让他们编写并运行 Consumer 代码，从你的 RocketMQ 服务接收响应消息：

java
复制代码
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.common.message.MessageExt;

public class MQConsumer {
    public static void main(String[] args) throws Exception {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("example_consumer_group");
        consumer.setNamesrvAddr("127.0.0.1:9876");
        consumer.subscribe("ResponseTopic", "*");

        consumer.registerMessageListener((MessageListenerConcurrently) (msgs, context) -> {
            for (MessageExt msg : msgs) {
                System.out.printf("Received response message: %s %n", new String(msg.getBody()));
            }
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        });

        consumer.start();
        System.out.printf("Consumer Started.%n");
    }
}
总结
通过提供清晰的配置和示例代码，你的同事或客户可以轻松地调用你的 RocketMQ 服务，发送消息并接收固定响应内容。这使得你的 RocketMQ 服务可以作为一个 mock 服务，为其他系统或服务提供模拟响应。
