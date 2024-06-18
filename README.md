import org.springframework.http.*;
import org.springframework.web.client.RestTemplate;
import org.yaml.snakeyaml.Yaml;
import org.yaml.snakeyaml.constructor.Constructor;

import java.util.*;
import java.util.stream.Collectors;

public class GitHubService {

    private final RestTemplate gitRestTemplateWithProxy;
    private final GitHubProperties gitHubProperties;

    public GitHubService(RestTemplate gitRestTemplateWithProxy, GitHubProperties gitHubProperties) {
        this.gitRestTemplateWithProxy = gitRestTemplateWithProxy;
        this.gitHubProperties = gitHubProperties;
    }

    public List<AAA> fetchYamlFile(String gitUrl_) {
        String gitUrl = constructApiUrl(gitUrl_);
        String gitRepoUrl = String.format("%s/contents/jdbc_list.yaml", gitUrl);
        HttpHeaders headers = new HttpHeaders();
        headers.setBearerAuth(gitHubProperties.getToken());
        HttpEntity<String> entity = new HttpEntity<>(headers);
        ResponseEntity<GithubFile[]> response = gitRestTemplateWithProxy.exchange(
                gitRepoUrl, HttpMethod.GET, entity, GithubFile[].class);
        
        if (response.getBody() == null || response.getBody().length == 0) {
            return Collections.emptyList();
        }

        String fileContent = new String(Base64.getDecoder().decode(response.getBody()[0].getContent()));
        Yaml yaml = new Yaml(new Constructor(Map.class));
        Map<String, String> yamlMap = yaml.load(fileContent);

        return yamlMap.entrySet().stream()
                .map(entry -> new AAA(entry.getKey(), entry.getValue()))
                .collect(Collectors.toList());
    }

    private String constructApiUrl(String gitUrl_) {
        // Implement the logic to construct the API URL
        // Assuming gitUrl_ is in the format of "<owner>/<repo>"
        return String.format("https://api.github.com/repos/%s", gitUrl_);
    }

    public static class GitHubProperties {
        private String token;

        public String getToken() {
            return token;
        }

        public void setToken(String token) {
            this.token = token;
        }
    }

    public static class GithubFile {
        private String name;
        private String content;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public String getContent() {
            return content;
        }

        public void setContent(String content) {
            this.content = content;
        }
    }

    public static class AAA {
        private String name;
        private String value;

        public AAA(String name, String value) {
            this.name = name;
            this.value = value;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public String getValue() {
            return value;
        }

        public void setValue(String value) {
            this.value = value;
        }
    }
}
