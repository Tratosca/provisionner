# What is CApen-SSL?

CApen-SSL (for Certification Autority OpenSSL) is a wrapper to facilitate the creation of a certification autority and server certificates using only OpenSSL commands.  
CApen-SSL works with a main script named build-certs. The build-certs script will wait for the input of values to build a certificate (examples below).  
CApen-SSL works in a way that first builds one root certificate autority, then the intermediate autority and finally server certificates.  
The CA scripts generated by build-certs will be named rootca.crt for the root autority and intca.crt for the intermediate autority. The server certificates will be named with the FQDN you give.

## Create manually own certificate autority

### Create the root certificate autority, example configuration

```openssl req -x509 -nodes -newkey rsa:4096 -days 3650 -extensions v3_ca -subj "/C=FR/ST=Ile-de-France/O=FictOrg/OU=Engineers/L=Paris/CN=FictOrg ROOT CA" -keyout out/ca/keys/rootca.key -out out/ca/certs/rootca.crt -config confs/ca.cnf```

### Create the private key for the intermediate certificate

```openssl genrsa -out out/ca/keys/intca.key 4096```

### Create the CSR to sign the intermediate CA with the root CA

```openssl req -sha256 -new -subj "/C=FR/ST=Ile-de-France/O=FictOrg/OU=Engineers/L=Paris/CN=FictOrg INT CA" -key out/ca/keys/intca.key -out out/ca/csr/intca.csr```

### Sign the CSR with the root certificate

```openssl ca -batch -config confs/ca.cnf -days 1825 -extensions v3_ca -subj "/C=FR/ST=Ile-de-France/O=FictOrg/OU=Engineers/L=Paris/CN=FictOrg INT CA" -notext -in out/ca/csr/intca.csr -out out/ca/certs/intca.crt```

### Create a private key for the server certificate

```openssl genrsa -out out/server/keys/test.test.lan.key 4096```

### Create the CSR with the intermediate certificate

```openssl req -new -key out/server/keys/test.test.lan.key -out out/server/csr/test.test.lan.csr -config confs/server.cnf```

### Sign the CSR with the intermediate certificate

```openssl x509 -req -in out/server/csr/test.test.lan.csr -CA out/ca/certs/intca.crt -CAkey out/ca/keys/intca.key -CAcreateserial -out out/server/certs/test.test.lan.crt -days 365 -sha512 -extfile confs/server.cnf```

## Use the build-certs script (BASH wrapper, recommended method)

### Root certificate autority (example values, but all below arguments have to be mentioned, in any order)
```./build-certs --country FR --province 'Ile-de-France' --locality Paris --organization FictOrg --unit Engineers --domain 'FicOrg ROOT CA' --days 3650 --root```

### Intermediate certification autority (example values, but all below arguments have to be mentioned, in any order)
```./build-certs --country FR --province 'Ile-de-France' --locality Paris --organization FictOrg --unit Engineers --domain 'FictOrg INT CA' --days 1825 --intermediate```

### Server certificate (example values, but all below arguments have to be mentioned, in any order)
```./build-certs --country FR --province 'Ile-de-France' --locality Paris --organization FictOrg --unit Engineers --domain 'test.test.lan' --days 365 --server```

### Client certificate (example values, but all below arguments have to be mentioned, in any order)
```./build-certs --country FR --province 'Ile-de-France' --locality Paris --organization FictOrg --unit Engineers --domain 'test.test.lan' --days 365 --client```

## Purge

### Clean all files generated by the wrapper and commands (certs, ca, csr and private keys)
```./build-certs --clean-files```

### Clean the database generated by the wrapper and commands
```./build-certs --clean-database```

### Clean the configuration generated by the wrapper (certificates informations)
```./build-certs --clean-confs```

## Credits
Kevin Chevreuil 2022-2023
GNU GPLv3
