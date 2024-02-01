步骤 1: 生成RSA公钥和私钥
这一步骤通常在你的开发环境之外完成，使用如OpenSSL等工具。但是，为了完整性，这里是生成密钥的一个快速指南（假设你已经完成这一步并将密钥保存在了资源文件中）。

步骤 2: 加载公钥和私钥
假设你已经将公钥和私钥保存为资源文件，例如public_key.pem和private_key.pem。

加载私钥（用于生成Token）
java
Copy code
import com.auth0.jwt.algorithms.Algorithm;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.security.KeyFactory;
import java.security.PrivateKey;
import java.security.spec.PKCS8EncodedKeySpec;
import java.util.Base64;

public PrivateKey loadPrivateKey(String filename) throws Exception {
    String key = new String(Files.readAllBytes(Paths.get(getClass().getClassLoader().getResource(filename).toURI())));
    
    String privateKeyPEM = key
        .replace("-----BEGIN PRIVATE KEY-----", "")
        .replaceAll(System.lineSeparator(), "")
        .replace("-----END PRIVATE KEY-----", "");
    
    byte[] encoded = Base64.getDecoder().decode(privateKeyPEM);

    KeyFactory keyFactory = KeyFactory.getInstance("RSA");
    PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(encoded);
    return keyFactory.generatePrivate(keySpec);
}
加载公钥（用于验证Token）
java
Copy code
import java.security.KeyFactory;
import java.security.PublicKey;
import java.security.spec.X509EncodedKeySpec;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.Base64;

public PublicKey loadPublicKey(String filename) throws Exception {
    String key = new String(Files.readAllBytes(Paths.get(getClass().getClassLoader().getResource(filename).toURI())));
    
    String publicKeyPEM = key
        .replace("-----BEGIN PUBLIC KEY-----", "")
        .replaceAll(System.lineSeparator(), "")
        .replace("-----END PUBLIC KEY-----", "");
    
    byte[] encoded = Base64.getDecoder().decode(publicKeyPEM);

    KeyFactory keyFactory = KeyFactory.getInstance("RSA");
    X509EncodedKeySpec keySpec = new X509EncodedKeySpec(encoded);
    return keyFactory.generatePublic(keySpec);
}
步骤 3: 生成JWT Token
使用加载的私钥生成JWT Token。

java
Copy code
import com.auth0.jwt.JWT;
import com.auth0.jwt.algorithms.Algorithm;
import java.util.Date;

public String generateTokenForUser(UserInfo userInfo, PrivateKey privateKey) throws Exception {
    long expirationTime = 1000 * 60 * 60; // 1 hour in milliseconds
    Date expireAt = new Date(System.currentTimeMillis() + expirationTime);

    Algorithm algorithm = Algorithm.RSA256(null, privateKey); // 注意：公钥参数在这里不是必需的，因为签名只需要私钥

    String token = JWT.create()
            .withSubject(userInfo.getEmail())
            .withExpiresAt(expireAt)
            .withClaim("displayName", userInfo.getDisplayName())
            .withClaim("employeeId", userInfo.getEmployeeId())
            .withClaim("adGroup", userInfo.getAdGroup().toString())
            .sign(algorithm);

    return token;
}
步骤 4: 验证JWT Token
使用加载的公钥验证JWT Token。

java
Copy code
import com.auth0.jwt.JWT;
import com.auth0.jwt.JWTVerifier;
import com.auth0.jwt.algorithms.Algorithm;
import com.auth0.jwt.interfaces.DecodedJWT;

public boolean verifyToken(String token, PublicKey publicKey) {
    try {
        Algorithm algorithm = Algorithm.RSA256(publicKey, null); // 使用公钥验证
        JWTVerifier verifier = JWT.require(algorithm).build(); //Reusable verifier instance
        DecodedJWT jwt = verifier.verify(token);
        return true; // Token验证成功
    } catch (Exception e){
        // Token验证失败
        return false;
    }
}
请注意，这些代码示例需要根据你的实际应用场景进行调整，特别是在处理异常和资源文件路径时。确保你的资源文件（公钥和私钥）位于src/main/resources目录下，以便于通过类加载器访问。

User
帮我写下命令生成公钥私钥一套
ChatGPT
生成RSA公钥和私钥的一种常见方法是使用openssl工具。以下是生成密钥对的步骤和相应的命令：

生成私钥
首先，生成一个RSA私钥：

bash
Copy code
openssl genpkey -algorithm RSA -out private_key.pem -pkeyopt rsa_keygen_bits:2048
这个命令将生成一个2048位的RSA私钥，并将其保存到文件private_key.pem中。

生成公钥
接下来，从私钥生成公钥：

bash
Copy code
openssl rsa -pubout -in private_key.pem -out public_key.pem
这个命令将读取private_key.pem文件中的私钥，并从中提取公钥，保存到public_key.pem文件中。

注意事项
确保在一个安全的环境中执行这些命令，尤其是在生成私钥时。私钥必须严格保密，任何未授权的访问都可能导致安全风险。
这些命令生成的私钥和公钥文件应该妥善保存，特别是私钥文件，应该放在一个安全且可靠的地方。
2048位的密钥长度通常被认为是安全的，但是你也可以选择更高的密钥长度，如4096位，来增加安全性。请注意，更长的密钥长度可能会导致更高的计算成本。
执行这些命令后，你将得到RSA密钥对，可以用于JWT的签名和验证过程。



import org.springframework.stereotype.Component;
import javax.annotation.PostConstruct;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.security.KeyFactory;
import java.security.PrivateKey;
import java.security.spec.PKCS8EncodedKeySpec;
import java.util.Base64;

@Component
public class KeyLoader {

    private PrivateKey privateKey;

    @PostConstruct
    private void init() {
        try {
            this.privateKey = loadPrivateKey("path/to/private_key.pem");
        } catch (Exception e) {
            // Handle exception: log error, rethrow, etc.
            throw new RuntimeException("Failed to load private key", e);
        }
    }

    private PrivateKey loadPrivateKey(String filename) throws Exception {
        String key = new String(Files.readAllBytes(Paths.get(getClass().getClassLoader().getResource(filename).toURI())));
        
        String privateKeyPEM = key
            .replace("-----BEGIN PRIVATE KEY-----", "")
            .replaceAll(System.lineSeparator(), "")
            .replace("-----END PRIVATE KEY-----", "");
        
        byte[] encoded = Base64.getDecoder().decode(privateKeyPEM);

        KeyFactory keyFactory = KeyFactory.getInstance("RSA");
        PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(encoded);
        return keyFactory.generatePrivate(keySpec);
    }

    public PrivateKey getPrivateKey() {
        return this.privateKey;
    }
}
import org.springframework.beans.factory.annotation.Autowired;

public class TokenService {

    @Autowired
    private KeyLoader keyLoader;

    public String generateToken() {
        // ...
        PrivateKey privateKey = keyLoader.getPrivateKey();
        // 使用privateKey生成Token
        // ...
    }
}

