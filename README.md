Jws<Claims> jwsClaims = Jwts.parserBuilder()
                    .setSigningKey(publicKey)
                    .build()
                    .parseClaimsJws(token);

            // 从Token中提取信息
            String email = jwsClaims.getBody().getSubject();
            String displayName = jwsClaims.getBody().get("displayName", String.class);
            String employeeId = jwsClaims.getBody().get("employeeId", String.class);
            
