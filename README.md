import java.io.InputStream;
import java.nio.charset.StandardCharsets;
import java.security.KeyFactory;
import java.security.PublicKey;
import java.security.spec.X509EncodedKeySpec;
import java.util.Base64;

public class PublicKeyReader {

    public static RSAPublicKey get(String filename) throws Exception {
        // 读取公钥文件内容
        InputStream is = PublicKeyReader.class.getClassLoader().getResourceAsStream(filename);
        if (is == null) {
            throw new IllegalArgumentException("Public key file not found: " + filename);
        }
        String publicKeyPEM = new String(is.readAllBytes(), StandardCharsets.UTF_8);

        // 清除公钥字符串中的头部和尾部标记
        publicKeyPEM = publicKeyPEM
                .replace("-----BEGIN PUBLIC KEY-----", "")
                .replace("-----END PUBLIC KEY-----", "")
                .replaceAll("\\s+", "");

        // 对公钥进行Base64解码
        byte[] encoded = Base64.getDecoder().decode(publicKeyPEM);

        // 生成公钥
        KeyFactory keyFactory = KeyFactory.getInstance("RSA");
        X509EncodedKeySpec keySpec = new X509EncodedKeySpec(encoded);
        return (RSAPublicKey) keyFactory.generatePublic(keySpec);
    }
}



