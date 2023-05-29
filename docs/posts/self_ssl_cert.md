---
template: post.html
title: "Self Signed SSL Certificate Generation."
date: 2022-11-12
authors:
  - akkupy
tags: SSL self-signed raspberrypi certificate letsencrypt ssl
image:
  src: /assets/self-signed-ssl.png
  add_to_post: yes
  class: crop-excerpt
---


# Self Signed SSL Cerificate

## Private CA and self signed certs that work with Chrome

:warning: :poop: :warning: This method is for testing and development only. The vast majority of users should not use this method, as it requires loading a cert on each of your devices, which is both error-prone and requires future maintenance. Instead, focus your energy on obtaining real certs via [Let's Encrypt](https://letsencrypt.org/getting-started/). This can even work if your vaultwarden instance is not on the public Internet ([[example|Running-a-private-vaultwarden-instance-with-Let's-Encrypt-certs]]).

:skull_and_crossbones: :skull_and_crossbones: :skull_and_crossbones: This method is not supported. Please do not open GitHub issues or post on the discussion forums asking about how to get this to work.

---

To get docker based websites working properly with self-signed certificates, Chrome needs the certificate to include the domain name in the alternative name field of the certificate.

Create a CA key (your own little on-premise Certificate Authority):
```
openssl genpkey -algorithm RSA -aes128 -out private-ca.key -outform PEM -pkeyopt rsa_keygen_bits:2048
```

Note: instead of `-aes128` you could also use the older `-des3`.

Create a CA certificate:
```
openssl req -x509 -new -nodes -sha256 -days 3650 -key private-ca.key -out self-signed-ca-cert.crt
```

Note: the `-nodes` argument prevents setting a pass-phrase for the private key (key pair) in a test/safe environment, otherwise you'll have to input the pass-phrase every time you start/restart the server.

NOTE: change 'website' to desired website name.

Create a website key:
```
openssl genpkey -algorithm RSA -out website.key -outform PEM -pkeyopt rsa_keygen_bits:2048
```

Create the website certificate request file:
```
openssl req -new -key website.key -out website.csr
```

Create a text file `website.ext` with the following content, change the domain names to your setup.
```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = website.local
DNS.2 = www.website.local
```


Create the website certificate, signed from the root CA:

```
openssl x509 -req -in website.csr -CA self-signed-ca-cert.crt -CAkey private-ca.key -CAcreateserial -out website.crt -days 365 -sha256 -extfile website.ext
```
Note: As of April 2019 iOS 13+ and macOS 15+, the server certificate can not have an expiry > 825 and must include ExtendedKeyUsage extension https://support.apple.com/en-us/HT210176 
 
## Add the root certificate and the website certificate to client computers.

Add the self-signed-ca-cert.crt generated to client web browsers.

Add website.crt,website.key to corresponding website or proxy manager.

## Reference

For reference, [see here](https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/)
