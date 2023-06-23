# RBAC Authorization

## To Add Subject Alternate Name (SAN)
```
[ req ]
distinguished_name  = req_distinguished_name
policy              = policy_match
x509_extensions     = user_crt
req_extensions      = v3_req
[ req_distinguished_name ]
countryName                     = Country Name (2 letter code)
countryName_default             = IN
countryName_min                 = 2
countryName_max                 = 2
stateOrProvinceName             = State or Province Name (full name)
stateOrProvinceName_default     = UNKNOWN
localityName                    = Locality Name (eg, city)
localityName_default            = UNKNOWN
0.organizationName              = Organization Name (eg, company)
0.organizationName_default      = UNKNOWN
organizationalUnitName          = Organizational Unit Name (eg, section)
organizationalUnitName_default  = UNKNOWN
commonName                      = Common Name
commonName_max                  = 64
emailAddress                    = Email Address
emailAddress_max                = 64
[ user_crt ]
nsCertType              = client, server, email
nsComment               = "OpenSSL Generated Certificate"
subjectKeyIdentifier    = hash
authorityKeyIdentifier  = keyid,issuer
[ v3_req ]
basicConstraints        = CA:FALSE
extendedKeyUsage        = serverAuth, clientAuth
subjectAltName          = @alt_names
[alt_names]
DNS.1 = ec2-44-203-168-246.compute-1.amazonaws.com
DNS.2 = ec2-52-90-93-217.compute-1.amazonaws.com
DNS.3 = ec2-54-175-225-159.compute-1.amazonaws.com
IP.1 = 127.0.0.
```

## First Create Client Key and CSR
```
openssl genrsa \
  -out client.key 2048

openssl req \
  -new \
  -key client.key \
  -config openssl.cnf \
  -out client.csr
```

### Get CSR signed by the Certificate Authority

## KeyStore and TrustStore for Client
```
# Create P12 file with client key and client certificate
openssl pkcs12 -export -in client.cer -inkey client.key -out client.p12

# Create KeyStore with CA certificate
keytool -keystore client.keystore.jks -import -file ca.cer

# Update KeyStore with client.p12 file
keytool -importkeystore \
        -deststorepass password \
        -destkeystore client.keystore.jks \
        -srckeystore client.p12 \
        -srcstoretype PKCS12 \
        -srcstorepass password

# Create TrustStore with CA certificate
keytool -import -file ca.cer -keystore truststore.jks
```

## Now Create client.properties
```
# Common configs for both producer and consumer
security.protocol=SSL
ssl.truststore.location=/Users/viveksinghania/Desktop/mTLS/truststore.jks
ssl.truststore.password=password

# Only needed if client authentication is required by the brokers
ssl.keystore.location=/Users/viveksinghania/Desktop/mTLS/client.keystore.jks
ssl.keystore.password=password
ssl.endpoint.identification.algorithm=
```
## Pre Requisite
```
Before creating Topic make sure Topic Assingnment is Provided
Principal type is User 
Principal name is same as Client Certificate CN Name.
Role as ResourceOwner (To have access to Create/Write/Read)
Pattern Type as Prefix/Literal (Prefix for WildCard)
ResourceID as t_vivek (Can have access for t_vivek topic)

To Read from Topic First need to Group Assingnment is Provided
Follow the above steps

```

## To connect to the Kafka server and create topic
```
# Create Topic
./bin/kafka-topics \
--bootstrap-server ec2-44-203-168-246.compute-1.amazonaws.com:9091 \
--create --topic t_vivek \
--command-config /root/clientProps/ssl.properties

# List the Topic
./bin/kafka-topics \
--bootstrap-server ec2-44-203-168-246.compute-1.amazonaws.com:9091 \ 
--list \
--command-config /root/clientProps/ssl.properties

# Produce Topic
./bin/kafka-console-producer \
--bootstrap-server ec2-44-203-168-246.compute-1.amazonaws.com:9091 \
--topic t_vivek \
--producer.config /root/clientProps/ssl.properties

# Consume from Topic
./bin/kafka-console-consumer \
--bootstrap-server ec2-44-203-168-246.compute-1.amazonaws.com:9091 \
--topic t_vivek --from-beginning \
--group vivek \
--consumer.config /root/clientProps/ssl.properties
```
