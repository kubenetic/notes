# How to create PKI for homelab

## Generate Root CA

As I write these lines I don't have the enthusiasm to create and maintain a database for the signed and revoked
certificates, it's just a homelab environment. 

First of all, I generated the RootCA's private key
```bash
openssl genrsa -out root_ca.key 4096
```

Then I created a config file for the `openssl` command with the following content.

```plain
# [ req ] section defines settings for creating a certificate request
[ req ]
# Specifies the distinguished_name section, which holds the X.509 attributes (DN fields) like 
# country, state, organization, etc. These fields will populate the certificate's subject.
distinguished_name = req_distinguished_name

# Specifies the x509 extensions to be applied to the self-signed certificate. 
# This indicates that the certificate being created will be a CA (Certificate Authority) 
# with CA-specific extensions (v3_ca).
x509_extensions = v3_ca

# This disables the prompt for user input. By setting `prompt = no`, OpenSSL will use the 
# values provided in the [req_distinguished_name] section without asking the user to fill in 
# details interactively. This makes the process repeatable and automatic.
prompt = no

# [ req_distinguished_name ] section defines the values for the certificate's distinguished name (DN),
# which will be embedded in the certificate. These fields identify the issuer of the certificate.
[ req_distinguished_name ]
# C is the country code. Here it is set to Belgium (BE).
C = BE

# ST is the state or province name. In this case, it’s set to Belgium.
ST = Belgium

# L is the locality, typically a city or town. Here it is set to Bruxelles (Brussels).
L = Bruxelles

# O is the organization name. This could be your company or homelab project name. 
# In this case, it's set to "Kubenetic".
O = Kubenetic

# CN stands for Common Name, which typically represents the hostname or identity of the certificate owner.
# In the case of a root CA, it's generally called something like "RootCA" to identify it as the top-level certificate.
CN = RootCA

# [ v3_ca ] section defines the X.509 v3 extensions specific to a CA certificate. 
# These extensions determine the certificate's functionality and role within the PKI hierarchy.
[ v3_ca ]
# The subjectKeyIdentifier extension is used to uniquely identify certificates issued by a particular CA. 
# 'hash' means it will use a hash of the public key as the identifier.
subjectKeyIdentifier = hash

# The authorityKeyIdentifier extension helps in linking a certificate to its issuing CA by including the
# key identifier of the issuer. 'keyid:always,issuer' means it will always include the key identifier and issuer 
# information.
authorityKeyIdentifier = keyid:always,issuer

# The basicConstraints extension is critical and specifies that this certificate is a CA certificate.
# 'CA:true' indicates that this certificate can sign other certificates (i.e., it can act as a CA).
# 'critical' ensures that this extension is strictly enforced when verifying the certificate.
basicConstraints = critical,CA:true

# The keyUsage extension defines what the certificate can be used for. 
# 'critical' means this extension is mandatory during verification.
# - 'digitalSignature': Allows the certificate to be used for signing (important for CA).
# - 'cRLSign': Allows the certificate to sign Certificate Revocation Lists (CRLs).
# - 'keyCertSign': Allows the certificate to sign other certificates (necessary for a CA).
keyUsage = critical, digitalSignature, cRLSign, keyCertSign
```

1. [ req ] Section:

    This section defines how OpenSSL behaves when creating a certificate request. The distinguished_name line points to the section that specifies the subject details (country, state, etc.), while x509_extensions specifies the extensions that will be applied to the certificate (for example, whether it will function as a CA). The prompt = no setting prevents OpenSSL from asking for interactive input, making the process fully automated with the predefined values.

2. [ req_distinguished_name ] Section:

    This defines the subject fields of the certificate. Each field corresponds to an attribute in the Distinguished Name (DN). The values given here will be embedded into the certificate and help identify the root CA in a PKI setup. The Common Name (CN) is particularly important because it identifies the certificate itself, in this case as a Root CA.

3. [ v3_ca ] Section:

    This section specifies the X.509 v3 extensions that make the certificate suitable for use as a CA. Extensions like basicConstraints, keyUsage, and authorityKeyIdentifier define the certificate’s role, restrict its use to CA tasks (like signing other certificates), and help ensure its authority can be traced.

Then I generated the RootCA's public key (lock).

```bash
openssl req -new -x509 -days 3650 -key root_ca.key -out root_ca.crt -config root_ca.cnf
```

After that I checked the generated certificate with the command below.

```bash
openssl x509 -in root_ca.crt -text -noout
```

## Generate Intermediate CA

I generated the private key for the intermedate ca with the same command as I did with the root CA. Then generate and
sign the certificate for the key in one command.

```bash
openssl genrsa -out intermediate_ca.key 4096
openssl req -nodes \
       -CA root_ca.crt -CAkey root_ca.key \ 
       -newkey rsa:4096 -keyout intermediate_ca.key \
       -x509 -days 3650  \
       -subj '/C=BE/ST=Belgium/L=Bruxelles/O=Kubenetic/CN=IntermediateCA/emailAddress=info@kubenetic.eu' \
       -out intermediate_ca.crt
```

## Generate server TLS keypairs

The method is just the same as I did here with the intermediate CA. In case of you want to add _subject alternative
names_ you can do it with the following command:

```bash
openssl req -nodes \
    -newkey rsa:2048 \
    -CA intermediate_ca.crt -CAkey intermediate_ca.key \
    -keyout sonar/server.key \
    -x509 -days 365 \
    -subj '/C=BE/ST=Belgium/L=Bruxelles/O=Kubenetic/CN=sonar.kubenetic.home/emailAddress=info@kubenetic.eu' \
    -addext 'subjectAltName = DNS:service, DNS:service.home.arpa \
    -out sonar/server.crt 
```

## Read more on topic:

* [Alternative subject names](https://security.stackexchange.com/questions/74345/provide-subjectaltname-to-openssl-directly-on-the-command-line)
* [OpenSSL essetials by DigitalOcean](https://www.digitalocean.com/community/tutorials/openssl-essentials-working-with-ssl-certificates-private-keys-and-csrs)
* [OpenSSL cheatsheet](https://www.golinuxcloud.com/openssl-cheatsheet/)



