private void updateAwsSecretsManager(String name, String jenkinsCredential, String mailAddress) {
        String secretName = "your-aws-secret-name";
        SecretsManagerAsyncClient client = awsService.getSecretsManagerAsyncClient();

        GetSecretValueRequest getSecretValueRequest = GetSecretValueRequest.builder()
                .secretId(secretName)
                .build();

        client.getSecretValue(getSecretValueRequest).thenAccept(result -> {
            try {
                String secretString = result.secretString();
                ObjectMapper mapper = new ObjectMapper();
                JsonNode secretJson = mapper.readTree(secretString);

                JsonNode jenkinsNode = secretJson.path("jenkins");
                if (jenkinsNode.isObject()) {
                    ((ObjectNode) jenkinsNode).putObject(name)
                            .put("user", mailAddress)
                            .put("token", jenkinsCredential);
                }

                PutSecretValueRequest putSecretValueRequest = PutSecretValueRequest.builder()
                        .secretId(secretName)
                        .secretString(mapper.writeValueAsString(secretJson))
                        .build();

                client.putSecretValue(putSecretValueRequest);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }).join();
    }
