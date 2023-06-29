# Distribute Connect Cluster

In this project, I built a robust, secure, distributed event-streaming system on Digital Ocean using two Linux 20.04 servers. The foundation was laid with Java installations on both servers, paving the way for the Confluent Platform, an extensive real-time data handling solution.

To fortify security, I implemented Role-Based Access Control (RBAC) for a Connect Worker, updating the connect-distributed.properties file. This grants fine-grained access and enhances security.

Further, I set up a logging mechanism by giving the system access to a specific directory, ensuring the smooth operation and expedited debugging. Logs provide valuable insights into system activities and aid in resolving issues quickly.

A significant security aspect was the addition of KeyStore and TrustStore. This empowers the system to connect securely to the broker using mutual TLS (mTLS), a security protocol ensuring mutual authentication between communicating parties.

Lastly, I created Role Bindings using the Confluent CLI. This determines user or application access levels to resources, a critical part of the RBAC model.

To summarize, this project encompassed the development of a secure, distributed event-streaming system that can efficiently manage real-time data while providing strong access controls.feeds while enforcing stringent access controls.

## Install Java
```
apt install openjdk-17-jre-headless
```

## Install Confluent Platform Kafka
```
wget -qO - https://packages.confluent.io/deb/7.4/archive.key | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://packages.confluent.io/deb/7.4 stable main"

sudo add-apt-repository "deb https://packages.confluent.io/clients/deb $(lsb_release -cs) main"

sudo apt-get update && sudo apt-get install confluent-platform
```

## Before Starting the kafka Connect
```
Change the confluent-kafka-connect.service file, 
To make Sure Connect-distributed.properties file is used to start the kafka connect.

cd /lib/systemd/system
vim confluent-kafka-connect.service

ExecStart=/usr/bin/connect-distributed /etc/kafka/connect-distributed.properties
```

## To start the kafka Connect and check status
```
sudo systemctl start confluent-kafka-connect

systemctl status confluent-kafka-connect

ps aux | grep 12453

# Check if connector is running
curl http://68.183.92.186:8083/connectors
```


## Make sure Server has KeyStore and TrustStore
```
# Optional, only if client authentication is needed
scp -i ./keygen ./mTLS/client.keystore.jks root@68.183.92.186:/macbookfile

scp -i ./keygen ./mTLS/truststore.jks root@68.183.92.186:/macbookfile
```


## Configure connect-distributed.properties for RBAC
```
cd /etc/kafka
vim connect-distributed.properties 

Edit:
bootstrap.servers=
group.id=
offset.storage.topic=
config.storage.topic=
status.storage.topic=

rest.advertised.host.name=host
rest.advertised.port=port
rest.advertised.listener=http/https

connector.client.config.override.policy=All

# Or SASL_SSL if using SSL
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
  username="username" \
  password="password";

ssl.truststore.location=/path/to/TrustStore/truststore.jks
ssl.truststore.password=password

# Optional, only if client authentication is needed
ssl.keystore.location=/path/to/keystore/keystore.jks
ssl.keystore.password=password

ssl.endpoint.identification.algorithm=

producer.security.protocol=SASL_SSL
producer.sasl.mechanism=PLAIN

consumer.security.protocol=SASL_SSL
consumer.sasl.mechanism=PLAIN

admin.security.protocol=SASL_SSL
admin.sasl.mechanism=PLAIN

# Adds the RBAC REST extension to the Connect worker
rest.extension.classes=io.confluent.connect.security.ConnectSecurityExtension

# The location of a running metadata service
confluent.metadata.bootstrap.server.urls=server:port

# Credentials to use when communicating with the MDS
confluent.metadata.basic.auth.user.info=username:password
confluent.metadata.http.auth.credentials.provider=BASIC
```

## Change permission to Write Logs
```
chmod -R 770 var
```

### If not Authorized you might see Error in Logs
```
cd /var/log/kafka
tail -f connect.log
``` 

## To Allow Access:

## From truststore get certificate
```
keytool -exportcert -keystore truststore.jks  -file certificate.crt
```

## Convert certificate to pem formate
```
openssl x509 -inform der -in certificate.crt -out certificate.pem
```

## Use Confluent CLI to Log-In as Admin to give permission
```
confluent login --url ec2-34-201-67-56.compute-1.amazonaws.com:8090 --ca-cert-path /macbookfile/certificate.pem
```

## Get the Cluster ID
```
curl https://ec2-34-201-67-56.compute-1.amazonaws.com:8090/kafka/v3/clusters -k -u username:password
```

## Confluent CLI Role-Bindings
```
List the available RBAC roles and associated information,
confluent iam rbac role-binding list -o human --kafka-cluster vlqJLaAsSe6hNV-eOQ_0lA --principal User:connectAdmin
```

## Create Role Binding
```
confluent iam rbac role-binding create --role ResourceOwner --resource Topic:connect-offsets-vivek --kafka-cluster vlqJLaAsSe6hNV-eOQ_0lA --principal User:connectAdmin

confluent iam rbac role-binding create --role ResourceOwner --resource Topic:connect-configs-vivek --kafka-cluster vlqJLaAsSe6hNV-eOQ_0lA --principal User:connectAdmin

confluent iam rbac role-binding create --role ResourceOwner --resource Topic:connect-status-vivek --kafka-cluster vlqJLaAsSe6hNV-eOQ_0lA --principal User:connectAdmin

confluent iam rbac role-binding create --role ResourceOwner --resource Group:connect-cluster-vivek --kafka-cluster vlqJLaAsSe6hNV-eOQ_0lA --principal User:connectAdmin
```

### Restart the Kafka Connect
```
sudo systemctl restart confluent-kafka-connect
```
