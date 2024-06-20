<!DOCTYPE html>
<html>
<head>
    <title>Jenkins Config Form</title>
</head>
<body>
    <div class="form-container" style="max-width: 500px; margin: auto; padding: 20px; border: 1px solid #ccc; border-radius: 10px; box-shadow: 2px 2px 12px #aaa;">
        <h2 style="text-align: center;">Jenkins Config Form</h2>
        <form id="jenkinsConfigForm">
            <div class="form-group" style="margin-bottom: 15px;">
                <label for="name" style="display: block; margin-bottom: 5px;">Name</label>
                <input type="text" id="name" name="name" required style="width: 100%; padding: 8px; box-sizing: border-box;">
            </div>
            <div class="form-group" style="margin-bottom: 15px;">
                <label for="jenkinsCredential" style="display: block; margin-bottom: 5px;">Jenkins Credential</label>
                <input type="text" id="jenkinsCredential" name="jenkinsCredential" required style="width: 100%; padding: 8px; box-sizing: border-box;">
            </div>
            <div class="form-group" style="margin-bottom: 15px;">
                <label for="mailAddress" style="display: block; margin-bottom: 5px;">Mail Address by Jenkins Credential</label>
                <input type="text" id="mailAddress" name="mailAddress" required style="width: 100%; padding: 8px; box-sizing: border-box;">
            </div>
            <button type="submit" style="padding: 10px 15px; background-color: #28a745; color: white; border: none; border-radius: 5px; cursor: pointer;">Submit</button>
        </form>
        <div id="result" style="margin-top: 20px; text-align: center;"></div>
    </div>

    <script>
        document.getElementById('jenkinsConfigForm').addEventListener('submit', function(event) {
            event.preventDefault();
            
            const name = document.getElementById('name').value;
            const jenkinsCredential = document.getElementById('jenkinsCredential').value;
            const mailAddress = document.getElementById('mailAddress').value;
            
            const data = {
                name: name,
                jenkinsCredential: jenkinsCredential,
                mailAddress: mailAddress
            };

            fetch('/submitConfig', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(data)
            })
            .then(response => response.text())
            .then(result => {
                document.getElementById('result').innerText = result;
            })
            .catch(error => {
                document.getElementById('result').innerText = 'Error: ' + error;
            });
        });
    </script>
</body>
</html>


import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import com.amazonaws.services.secretsmanager.*;
import com.amazonaws.services.secretsmanager.model.*;
import com.google.gson.JsonObject;
import com.google.gson.JsonParser;

@RestController
public class JenkinsConfigController {

    @Autowired
    private JenkinsConfigRepository jenkinsConfigRepository;

    @PostMapping("/submitConfig")
    public String submitConfig(@RequestBody JenkinsConfigRequest request) {
        String name = request.getName();
        String jenkinsCredential = request.getJenkinsCredential();
        String mailAddress = request.getMailAddress();

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

    static class JenkinsConfigRequest {
        private String name;
        private String jenkinsCredential;
        private String mailAddress;

        // Getters and setters
        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public String getJenkinsCredential() {
            return jenkinsCredential;
        }

        public void setJenkinsCredential(String jenkinsCredential) {
            this.jenkinsCredential = jenkinsCredential;
        }

        public String getMailAddress() {
            return mailAddress;
        }

        public void setMailAddress(String mailAddress) {
            this.mailAddress = mailAddress;
        }
    }
}

