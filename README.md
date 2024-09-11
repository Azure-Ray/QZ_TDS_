@SpringBootTest
@EmbeddedKafka(
    partitions = 1,
    topics = {"test-topic"},
    brokerProperties = {
        "listeners=PLAINTEXT://localhost:9092,PLAINTEXT2://localhost:9093,PLAINTEXT3://localhost:9094",
        "advertised.listeners=PLAINTEXT://msk-poc-n1b-2be5c5c0cea41a73.elb.ap-east-1.amazonaws.com:9001,PLAINTEXT2://msk-poc-n1b-2be5c5c0cea41a73.elb.ap-east-1.amazonaws.com:9002,PLAINTEXT3://msk-poc-n1b-2be5c5c0cea41a73.elb.ap-east-1.amazonaws.com:9003",
        "listener.security.protocol.map=PLAINTEXT:PLAINTEXT,PLAINTEXT2:PLAINTEXT,PLAINTEXT3:PLAINTEXT"
    }
)
@TestPropertySource(properties = {
        "spring.kafka.bootstrap-servers=${spring.embedded.kafka.brokers}"
})
public class KafkaMqClientProviderTest {
    // 测试类代码

    @Autowired
    private EmbeddedKafkaBroker embeddedKafka;

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    private BlockingQueue<ConsumerRecord<String, String>> records;

    @BeforeEach
    public void setUp() {
        records = new LinkedBlockingQueue<>();

        // 设置消费者配置
        Map<String, Object> consumerProps = KafkaTestUtils.consumerProps
