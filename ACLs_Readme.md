# ACLs Authorization

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

# Now Update server.properties file
```
# The location of the key store file.
ssl.keystore.location=/Users/viveksinghania/Desktop/mTLS/server.keystore.jks
# The store password for the key store file.
ssl.keystore.password=password
# The password of the private key in the key store file.
#ssl.key.password=your-key-password

# The location of the trust store file.
ssl.truststore.location=/Users/viveksinghania/Desktop/mTLS/truststore.jks
# The password for the trust store file.
ssl.truststore.password=password

ssl.client.auth=required

# Super User Name
super.users=User:vivek

# Add ACLs Authorization
authorizer.class.name=kafka.security.authorizer.AclAuthorizer

# To get TLS/SSL principal user names
ssl.principal.mapping.rules=RULE:^.*[Cc][Nn]=([a-zA-Z0-9.]*).*$/$1/L,DEFAULT
```

