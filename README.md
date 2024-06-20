CREATE TABLE jenkins_config (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    type VARCHAR(255),
    platform VARCHAR(255),
    host VARCHAR(255),
    query TEXT
);
@Entity
@Table(name = "jenkins_config")
public class JenkinsConfigEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String type;
    private String platform;
    private String host;
    private String query;

    // Getters and setters
}

public interface JenkinsConfigRepository extends JpaRepository<JenkinsConfigEntity, Long> {
}
@Service
public class JenkinsConfigService {
    @Autowired
    private JenkinsConfigRepository jenkinsConfigRepository;

    public List<SplunkJenkinsConfig.JenkinsConfig> loadJenkinsConfigs() {
        List<JenkinsConfigEntity> entities = jenkinsConfigRepository.findAll();
        return entities.stream().map(this::convertToJenkinsConfig).collect(Collectors.toList());
    }

    private SplunkJenkinsConfig.JenkinsConfig convertToJenkinsConfig(JenkinsConfigEntity entity) {
        SplunkJenkinsConfig.JenkinsConfig config = new SplunkJenkinsConfig.JenkinsConfig();
        config.setName(entity.getName());
        config.setType(entity.getType());
        config.setPlatform(entity.getPlatform());
        config.setHost(entity.getHost());
        config.setQuery(entity.getQuery());
        return config;
    }
}
@Configuration
public class SplunkJenkinsConfig {
    @Getter
    private List<JenkinsConfig> splunkDetails = new ArrayList<>();

    @Autowired
    public SplunkJenkinsConfig(JenkinsConfigService jenkinsConfigService) {
        this.splunkDetails = jenkinsConfigService.loadJenkinsConfigs();
    }

    @Data
    public static class JenkinsConfig {
        private String name; 
        private String type;
        private String platform; 
        private String host; 
        private String query;
    }
}
