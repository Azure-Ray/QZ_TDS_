import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import software.amazon.awssdk.auth.credentials.DefaultCredentialsProvider;
import software.amazon.awssdk.core.ResponseInputStream;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.s3.S3Client;
import software.amazon.awssdk.services.s3.model.*;

import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.nio.charset.StandardCharsets;
import java.util.List;
import java.util.stream.Collectors;

@Service
public class S3Service {

    private final S3Client s3Client;
    private final String bucketName;

    public S3Service(@Value("${aws.s3.bucket-name}") String bucketName) {
        this.s3Client = S3Client.builder()
                .region(Region.of("your-region"))
                .credentialsProvider(DefaultCredentialsProvider.create())
                .build();
        this.bucketName = bucketName;
    }

    public List<String> listObjects() {
        ListObjectsV2Response response = s3Client.listObjectsV2(ListObjectsV2Request.builder().bucket(bucketName).build());
        return response.contents().stream().map(S3Object::key).collect(Collectors.toList());
    }

    public String readObjectContent(String key) {
        GetObjectRequest getObjectRequest = GetObjectRequest.builder().bucket(bucketName).key(key).build();
        StringBuilder resultStringBuilder = new StringBuilder();
        try (ResponseInputStream<GetObjectResponse> s3Object = s3Client.getObject(getObjectRequest)) {
            new BufferedReader(new InputStreamReader(s3Object, StandardCharsets.UTF_8))
                    .lines().forEach(resultStringBuilder::append);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return resultStringBuilder.toString();
    }

    public void uploadObject(String key, InputStream data, long size) {
        PutObjectRequest putObjectRequest = PutObjectRequest.builder()
                .bucket(bucketName)
                .key(key)
                .build();
        s3Client.putObject(putObjectRequest, software.amazon.awssdk.core.sync.RequestBody.fromInputStream(data, size));
    }

    public void deleteObject(String key) {
        DeleteObjectRequest deleteObjectRequest = DeleteObjectRequest.builder()
                .bucket(bucketName)
                .key(key)
                .build();
        s3Client.deleteObject(deleteObjectRequest);
    }

    public void copyObject(String sourceKey, String destinationKey) {
        CopyObjectRequest copyObjectRequest = CopyObjectRequest.builder()
                .sourceBucket(bucketName)
                .sourceKey(sourceKey)
                .destinationBucket(bucketName)
                .destinationKey(destinationKey)
                .build();
        s3Client.copyObject(copyObjectRequest);
    }

    // 这里可以继续添加其他你需要的S3服务方法
}
