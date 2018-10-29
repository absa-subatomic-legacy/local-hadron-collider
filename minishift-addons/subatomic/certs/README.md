# Generating local Subatomic certificates

To mimic as real an environment as possible, we want to use TLS wherever possible.
This forces certificate related issues to be picked up and solved as early in the development lifecycle as possible.

The below guide explains how to generate the necessary certificates to allow applications to use secured connections.
This includes a Subatomic root certificate that can be trusted locally so that browsers and other tools verify the Subatomic certificates.

The following steps were adapted from this guide: https://gist.github.com/Soarez/9688998

## Generating a Subatomic root certificate

We need our own root CA certificate to sign our certificates. This allows us to trust this root certificate so that browsers and other tools can trust/verify our certificates signed by this root certificate.

### Creating a `ssl.conf`

First we need a [`ssl.conf`](ssl.conf) file that will contain some common `openssl` configuration options to dave some typing later.

```
[ req ]
default_bits       = 4096
distinguished_name = req_distinguished_name
req_extensions     = req_ext

[ req_distinguished_name ]
countryName                 = Country Name (2 letter code)
countryName_default         = ZA
stateOrProvinceName         = State or Province Name (full name)
stateOrProvinceName_default = Gauteng
localityName                = Locality Name (eg, city)
localityName_default        = Johannesburg
organizationName            = Organization Name (eg, company)
organizationName_default    = Subatomic
commonName                  = Common Name (e.g. server FQDN or YOUR name)
commonName_max              = 64
commonName_default          = subatomic.local

[ req_ext ]
subjectAltName = @alt_names

[alt_names]
DNS.1   = *.subatomic.local
```

not the Subject Alternate name section and remember to adjust if necessary.

### Create the Subatomic root certificate secret and Certificate Signing Request

```console
$ openssl genrsa -out subatomic.ca.key 2048
$ openssl req -new -key subatomic.ca.key -out subatomic.ca.csr
```

Now you are ready to create the root certificate with the following:

```console
$ openssl req -new -x509 -days 3650 -key subatomic.ca.key -out subatomic.ca.crt -config ssl.conf
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [ZA]:
State or Province Name (full name) [Gauteng]:
Locality Name (eg, city) [Johannesburg]:
Organization Name (eg, company) [Subatomic]:
Common Name (e.g. server FQDN or YOUR name) [subatomic.local]:ca.subatomic.local
```

note the expiry is set 10 years into the future with `-days 3650`.

> ⚠️ Do not set the Common Name of your root certificate to the same as your domain. I.e. don't use a Common Name of `subatomic.local` but instead use something like `ca.subatomic.local`. You will get TLS verification issues if they are the same.

## Generating *.subatomic.local wildcard certificate

For simplicity we can create a wildcard certificate that'll cover all `subatomic.local` FQDNs.
Do this with the following:

```console
$ openssl x509 -days 3650 -req -in subatomic.local.csr -CA subatomic.ca.crt -CAkey subatomic.ca.key -CAcreateserial -out subatomic.local.crt -extensions req_ext -sha256 -extfile ssl.conf
Signature ok
subject=/C=ZA/ST=Gauteng/L=Johannesburg/O=Subatomic/CN=subatomic.local
Getting CA Private Key
```

## Trusting the Subatomic root certificate

On a Mac you can install and trust the Subatomic root certificate with:

```console
$ sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain subatomic.ca.crt
```

or follow the many guides on doing this through the Keychain Access application.