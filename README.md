String hardcodedPublicKeyString = "YOUR HARDCODED PUBLIC KEY STRING HERE";
        try {
            publicKey = (RSAPublicKey) KeyFactory.getInstance("RSA")
                        .generatePublic(new X509EncodedKeySpec(Base64.getDecoder().decode(hardcodedPublicKeyString)));
        } catch (Exception ex) {
            Log.error("Error initializing hardcoded public key", ex.getMessage());
            return null; // or handle this case as per your application's need
        }
