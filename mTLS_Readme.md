# mTLS Connection

This Readme if for the mTLS connection between Kafka and Client

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



## Create Key and CSR for Server
```
openssl genrsa \
  -out server.key 2048

openssl req \
  -new \
  -key server.key \
  -config openssl.cnf \
  -out server.csr
```

## Create Key and CSR for Client
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


## KeyStore and TrustStore for Server
```
# Create P12 file with server key and server certificate
openssl pkcs12 -export -in server.cer -inkey server.key -out server.p12

# Create KeyStore with CA certificate
keytool -keystore server.keystore.jks -import -file ca.cer

# Update KeyStore with server.p12 file
keytool -importkeystore \
        -deststorepass password \
        -destkeystore server.keystore.jks \
        -srckeystore server.p12 \
        -srcstoretype PKCS12 \
        -srcstorepass password

# Create TrustStore with CA certificate
keytool -import -file ca.cer -keystore truststore.jks
```

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

## To connect to the Server from Client using SSH
```
ssh -i /path/to/your/private/key username@server-ip-address
```
