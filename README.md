# ApacheKafka
Apache Kafka commands

Start Zookeeper

Windows: zookeeper-server-start.bat ..\..\config\zookeeper.properties
Mac: ./zookeeper-server-start.sh ../config/zookeeper.properties

Start Kafka broker

Windows: kafka-server-start.bat ..\..\config\server.properties
Mac: ./kafka-server-start.sh ../config/server.properties

Create a Topic
Windows: kafka-topics.bat --create --topic <topic-name> -zookeeper localhost:2181 --replication-factor 1 --partitions 1.
Mac: ./kafka-topics.sh --create --topic <topic-name> -zookeeper localhost:2181 --replication-factor 1 --partitions 1
Example- ./kafka-topics.sh --create --topic my-first-topic -zookeeper localhost:2181 --replication-factor 1 --partitions 1

Check configuration of all the topics in the broker
Windows: kafka-topics.bat --describe --zookeeper localhost:2181
Mac: ./kafka-topics.sh --describe --zookeeper localhost:2181

Check configuration of a particular topic
Windows: kafka-topics.bat --describe --topic replicate_topic --zookeeper localhost:2181
Mac: ./kafka-topics.sh --describe --topic replicate_topic --zookeeper localhost:2181

Instaniate a console consumer
Windows: kafka-console-producer.bat --broker-list localhost:9092 --topic <topic-name>
Mac: ./kafka-console-consumer.sh --zookeeper localhost:2181 --topic <topic-name> --from-beginning
Example- ./kafka-console-consumer.sh --zookeeper localhost:2181 --topic my-first-topic --from-beginning

Delete a topic
Windows: kafka-topics.bat --delete --zookeeper localhost:2181 --topic your_topic_name
Mac: ./kafka-topics.sh --delete --zookeeper localhost:2181 --topic your_topic_name

Alter the configuration of a topic
Windows:
kafka-topics.bat --zookeeper localhost:2181 --alter --topic <topic-name> --partitions 4
kafka-topics.bat --zookeeper localhost:2181 --alter --topic demo-topic --partitions 4
Mac:
./kafka-topics.sh --zookeeper localhost:2181 --alter --topic <topic-name> --partitions 4
./kafka-topics.sh --zookeeper localhost:2181 --alter --topic demo-topic --partitions 4

Enable security in Kafka using SSL
SSL Set up in KAFKA BROKER:
Step 1:
Generate SSL key and Certificate for broker:
Keystore -> which stores each machine’s own identity.
Here we are creating the keystore file server.keystore.jks that stores the Certificate.
The validity of Certificate is given as 365 days below.
keytool -keystore server.keystore.jks -alias localhost -validity 365 -genkey

Step 2:
Creating your own CA:
Here we are creating a Certificate Authority which is responsible for signing certificates.
We will add these certs to the server.keystore.jks file and client.truststore.jks that we will be creating in a while.
openssl req -new -x509 -keyout ca-key -out ca-cert -days 365

Step 3:
Here we will generate the truststore
keytool -keystore server.truststore.jks -alias CARoot -import -file ca-cert

Step 4:
Sign all certificates in the keystore with the CA we generated.
Export the certificate in to the keystore.
keytool -keystore server.keystore.jks -alias localhost -certreq -file cert-file
Then sign it with the CA:
openssl x509 -req -CA ca-cert -CAkey ca-key -in cert-file -out cert-signed -days 365 -CAcreateserial -passin pass:kafka123
keytool -keystore server.keystore.jks -alias CARoot -import -file ca-cert
keytool -keystore server.keystore.jks -alias localhost -import -file cert-signed

Step 5:
Add the SSL in server.properties file of Kafka distribution
The below setting will make sure that the broker will authenticate the clients (Kafka Consumers) who are trying to access the broker.
listeners=SSL://localhost:9092
advertised.listeners=SSL://localhost:9092

security.inter.broker.protocol = SSL
ssl.client.auth=required

ssl.keystore.location=<path>/server.keystore.jks
ssl.keystore.password=changeit
ssl.key.password=changeit
ssl.truststore.location=<path>/server.truststore.jks
ssl.truststore.password=changeit
ssl.keystore.type = JKS
ssl.truststore.type = JKS

Step 6:
Run the below command to check servers keystore and truststore are set up correctly.
openssl s_client -debug -connect localhost:9093 -tls1
With this we came to the ends of Setting up the SSL in Kafka Broker.
Kafka Console Producer and Consumer using SSL:
Console Producer:
./kafka-console-producer.sh --broker-list localhost:9092 --topic test --producer.config ../client-ssl.properties
Console Consumer:
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning --new-consumer --consumer.config ../client-ssl.properties

Kill the Broker Process
Step 1:
ps ax | grep -i 'kafka\.Kafka'

Step 2:
kill -9 <processId>
