kafka-topics.bat --bootstrap-server msk-broker1.amazonaws.com:9001,msk-broker2.amazonaws.com:9002 --list
kafka-acls.bat --authorizer-properties zookeeper.connect=msk-broker1.amazonaws.com:2181 --add --allow-principal User:your-username --operation Read --topic *
kafka-topics.bat --bootstrap-server msk-broker1.amazonaws.com:9001,msk-broker2.amazonaws.com:9002 --list --command-config client-config.properties
security.protocol=SSL
ssl.truststore.location=/path/to/truststore.jks
ssl.truststore.password=your-password
