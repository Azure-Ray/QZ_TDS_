package com.example.awsdbtoken;

import com.amazonaws.auth.AWSStaticCredentialsProvider;
import com.amazonaws.auth.BasicAWSCredentials;
import com.amazonaws.auth.STSAssumeRoleSessionCredentialsProvider;
import com.amazonaws.services.rds.auth.RdsIamAuthTokenGenerator;
import com.amazonaws.services.securitytoken.AWSSecurityTokenService;
import com.amazonaws.services.securitytoken.AWSSecurityTokenServiceClientBuilder;
import com.amazonaws.services.securitytoken.model.AssumeRoleRequest;
import com.amazonaws.services.securitytoken.model.AssumeRoleResult;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/aws")
public class AwsDbAuthTokenController {

    @PostMapping("/generateToken")
    public String generateDbAuthToken(@RequestBody TokenRequest tokenRequest) {
        // Step 1: 使用用户传入的Access Key和Secret Key登录AWS
        BasicAWSCredentials awsCreds = new BasicAWSCredentials(tokenRequest.getAccessKey(), tokenRequest.getSecretKey());
        AWSSecurityTokenService stsClient = AWSSecurityTokenServiceClientBuilder.standard()
                .withRegion(tokenRequest.getRegion())
                .withCredentials(new AWSStaticCredentialsProvider(awsCreds))
                .build();

        // Step 2: Assume到指定的RDS IAM Role
        AssumeRoleRequest assumeRoleRequest = new AssumeRoleRequest()
                .withRoleArn(tokenRequest.getAssumeRoleArn())
                .withRoleSessionName("session" + System.currentTimeMillis());

        AssumeRoleResult assumeRoleResult = stsClient.assumeRole(assumeRoleRequest);
        
        // Step 3: 获取Assume Role的临时凭证
        STSAssumeRoleSessionCredentialsProvider credentialsProvider = new STSAssumeRoleSessionCredentialsProvider
                .Builder(tokenRequest.getAssumeRoleArn(), "session" + System.currentTimeMillis())
                .withStsClient(stsClient)
                .build();

        // Step 4: 使用临时凭证生成RDS认证Token
        RdsIamAuthTokenGenerator tokenGenerator = RdsIamAuthTokenGenerator.builder()
                .credentials(credentialsProvider)
                .region(tokenRequest.getRegion())
                .build();

        // 使用直接生成的Token请求参数
        String authToken = tokenGenerator.getAuthToken(
                tokenRequest.getHostname(), 3306, tokenRequest.getUsername()
        );

        return authToken;
    }
}

// 定义请求体模型
class TokenRequest {
    private String accessKey;
    private String secretKey;
    private String region;
    private String assumeRoleArn;
    private String hostname;
    private String username;

    // Getters and Setters
    public String getAccessKey() { return accessKey; }
    public void setAccessKey(String accessKey) { this.accessKey = accessKey; }

    public String getSecretKey() { return secretKey; }
    public void setSecretKey(String secretKey) { this.secretKey = secretKey; }

    public String getRegion() { return region; }
    public void setRegion(String region) { this.region = region; }

    public String getAssumeRoleArn() { return assumeRoleArn; }
    public void setAssumeRoleArn(String assumeRoleArn) { this.assumeRoleArn = assumeRoleArn; }

    public String getHostname() { return hostname; }
    public void setHostname(String hostname) { this.hostname = hostname; }

    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
}
