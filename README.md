package com.example.awsdbtoken;

import software.amazon.awssdk.auth.credentials.AwsBasicCredentials;
import software.amazon.awssdk.auth.credentials.AwsCredentialsProvider;
import software.amazon.awssdk.auth.credentials.AssumeRoleRequest;
import software.amazon.awssdk.auth.credentials.StsAssumeRoleCredentialsProvider;
import software.amazon.awssdk.auth.credentials.StaticCredentialsProvider;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.rds.RdsUtilities;
import software.amazon.awssdk.services.rds.model.GenerateAuthenticationTokenRequest;
import software.amazon.awssdk.services.sts.StsClient;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/aws")
public class AwsDbAuthTokenController {

    @PostMapping("/generateToken")
    public String generateDbAuthToken(@RequestBody TokenRequest tokenRequest) {
        // Step 1: 使用提供的 AWS Access Key 和 Secret Key 登录
        AwsCredentialsProvider staticProvider = StaticCredentialsProvider.create(
                AwsBasicCredentials.create(tokenRequest.getAccessKey(), tokenRequest.getSecretKey())
        );

        StsClient stsClient = StsClient.builder()
                .region(Region.of(tokenRequest.getRegion()))
                .credentialsProvider(staticProvider)
                .build();

        // Step 2: Assume到指定的 RDS IAM Role
        AssumeRoleRequest assumeRoleRequest = AssumeRoleRequest.builder()
                .roleArn(tokenRequest.getAssumeRoleArn())
                .roleSessionName("session" + System.currentTimeMillis())
                .build();

        StsAssumeRoleCredentialsProvider credentialsProvider = StsAssumeRoleCredentialsProvider.builder()
                .stsClient(stsClient)
                .refreshRequest(assumeRoleRequest)
                .build();

        // Step 3: 使用 Assume Role 后的临时凭证生成 RDS 认证 Token
        RdsUtilities rdsUtilities = RdsUtilities.builder()
                .region(Region.of(tokenRequest.getRegion()))
                .credentialsProvider(credentialsProvider)
                .build();

        String authToken = rdsUtilities.generateAuthenticationToken(
                GenerateAuthenticationTokenRequest.builder()
                        .hostname(tokenRequest.getHostname())
                        .port(3306) // 替换为实际端口
                        .username(tokenRequest.getUsername())
                        .build()
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


<dependencies>
    <!-- AWS SDK v2 Core -->
    <dependency>
        <groupId>software.amazon.awssdk</groupId>
        <artifactId>sts</artifactId>
        <version>2.20.15</version> <!-- 使用最新的 v2 版本 -->
    </dependency>
    <dependency>
        <groupId>software.amazon.awssdk</groupId>
        <artifactId>rds</artifactId>
        <version>2.20.15</version>
    </dependency>

    <!-- Spring Boot Starter Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
