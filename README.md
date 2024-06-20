<!DOCTYPE html>
<html>
<head>
    <title>Jenkins Config Form</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 40px;
        }
        .form-container {
            max-width: 500px;
            margin: auto;
            padding: 20px;
            border: 1px solid #ccc;
            border-radius: 10px;
            box-shadow: 2px 2px 12px #aaa;
        }
        .form-group {
            margin-bottom: 15px;
        }
        label {
            display: block;
            margin-bottom: 5px;
        }
        input[type="text"] {
            width: 100%;
            padding: 8px;
            box-sizing: border-box;
        }
        button {
            padding: 10px 15px;
            background-color: #28a745;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        button:hover {
            background-color: #218838;
        }
    </style>
</head>
<body>
    <div class="form-container">
        <h2>Jenkins Config Form</h2>
        <form id="jenkinsConfigForm" action="/submitConfig" method="post">
            <div class="form-group">
                <label for="name">Name</label>
                <input type="text" id="name" name="name" required>
            </div>
            <div class="form-group">
                <label for="jenkinsCredential">Jenkins Credential</label>
                <input type="text" id="jenkinsCredential" name="jenkinsCredential" required>
            </div>
            <div class="form-group">
                <label for="mailAddress">Mail Address by Jenkins Credential</label>
                <input type="text" id="mailAddress" name="mailAddress" required>
            </div>
            <button type="submit">Submit</button>
        </form>
    </div>
</body>
</html>



import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import com.amazonaws.services.secretsmanager.*;
import com.amazonaws.services.secretsmanager.model.*;

@RestController
public class JenkinsConfigController {

    @Autowired
    private JenkinsConfigRepository jenkinsConfigRepository;

    @PostMapping("/submitConfig")
    public String submitConfig(@RequestParam String name,
                               @RequestParam String jenkinsCredential,
                               @RequestParam String mailAddress) {
        String platform = "shp";
        String host = "https://jenkins-custom-" + name + ".digital-tools.enw1.prod.aws.cloud.hsbc";

        // 插入 build 类型的记录
        JenkinsConfigEntity buildConfig = new JenkinsConfigEntity();
        buildConfig.setName(name);
        buildConfig.setType("build");
        buildConfig.setPlatform(platform);
        buildConfig.setHost(host);
        buildConfig.setQuery(""); // 根据实际情况设置query字段
        jenkinsConfigRepository.save(buildConfig);

        // 插入 deploy 类型的记录
        JenkinsConfigEntity deployConfig = new JenkinsConfigEntity();
        deployConfig.setName(name);
        deployConfig.setType("deploy");
        deployConfig.setPlatform(platform);
        deployConfig.setHost(host);
        deployConfig.setQuery(""); // 根据实际情况设置query字段
        jenkinsConfigRepository.save(deployConfig);

        // 更新 AWS Secrets Manager
        updateAwsSecretsManager(name, jenkinsCredential, mailAddress);

        return "Config submitted successfully!";
    }

    private void updateAwsSecretsManager(String name, String jenkinsCredential, String mailAddress) {
        String secretName = "your-aws-secret-name";
        AWSSecretsManager client = AWSSecretsManagerClientBuilder.standard().withRegion("your-region").build();
        
        GetSecretValueRequest getSecretValueRequest = new GetSecretValueRequest().withSecretId(secretName);
        GetSecretValueResult getSecretValueResult = client.getSecretValue(getSecretValueRequest);

        String secretString = getSecretValueResult.getSecretString();
        JsonObject secretJson = new JsonParser().parse(secretString).getAsJsonObject();
        
        JsonObject jenkins = secretJson.getAsJsonObject("jenkins");
        JsonObject newCredential = new JsonObject();
        newCredential.addProperty("user", mailAddress);
        newCredential.addProperty("token", jenkinsCredential);
        
        jenkins.add(name, newCredential);
        secretJson.add("jenkins", jenkins);

        PutSecretValueRequest putSecretValueRequest = new PutSecretValueRequest()
                .withSecretId(secretName)
                .withSecretString(secretJson.toString());
        client.putSecretValue(putSecretValueRequest);
    }
}

<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-java-sdk-secretsmanager</artifactId>
    <version>1.11.828</version>
</dependency>
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.6</version>
</dependency>


