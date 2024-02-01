// Also adding a token to a Cookie
                String token = generateTokenForUser(userInfo); // Implement this method to generate or obtain a token
                ResponseCookie cookie = ResponseCookie.from("auth_token", token)
                        .httpOnly(true)
                        .secure(true)
                        .path("/")
                        .build();
                exchange.getResponse().addCookie(cookie);

                <dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.18.2</version> <!-- Use the latest version -->
</dependency>
import com.auth0.jwt.JWT;
import com.auth0.jwt.algorithms.Algorithm;
import java.util.Date;

private String generateTokenForUser(UserInfo userInfo) {
    // Token expiration time
    long expirationTime = 1000 * 60 * 60; // 1 hour in milliseconds
    Date expireAt = new Date(System.currentTimeMillis() + expirationTime);

    // Secret key used to sign the token. In a real application, it should be complex and stored securely.
    String secretKey = "your-secret-key";

    // Generate the token
    String token = JWT.create()
            .withSubject(userInfo.getEmail()) // Use user email as subject
            .withExpiresAt(expireAt)
            .withClaim("displayName", userInfo.getDisplayName())
            .withClaim("employeeId", userInfo.getEmployeeId())
            .withClaim("adGroup", userInfo.getAdGroup().toString()) // Assuming getAdGroup() returns a List or similar
            .sign(Algorithm.HMAC256(secretKey));

    return token;
}
