---
weight: 1
title: "Ch 4: SSL Certs"
bookToc: true
type: "docs"
bookFlatSection: true
---

# SSL Certs

Future services on my box will require [TLS](https://www.cloudflare.com/learning/ssl/transport-layer-security-tls/) (Transport Layer Security).

For this, I'll need to generate some SSL certificates.

## OpenSSL

[This free, open-source tool](https://github.com/openssl/openssl) came with my Debian install.

It'll work, though its hard to remember all the steps.

I follow along with a great youtube tutorial [here](https://www.youtube.com/watch?v=VH4gXcvkmOY).

## Certificate Authority 

CAs (certificate authorities) vouch for the identity / ownership of SSL certs.

I can create my own CA and sign my own certs for my private LAN.

### CA Private Key

I start by generating a private key for my CA.

It will be encrypted with aes256 and require a password each time I want to use it to sign something.

![ca private keygen](/ssl/ca_private_key.png)

### CA Cert

Using the private key, I then generate a cert for my CA.

For convenience, I set the expiration to 10 years.

![ca cert created](/ssl/ca_cert.png)

## Server Cert & Key

Now that I have a CA, I can set up the SSL cert and private key for my homelab server.

### Private Key

This is done with `sudo openssl genrsa -out cert-key.pem 4096`

### Cert Sign Request

Next I create a signing request with: 

`sudo openssl req -new -sha256 -subj "/CN=homelab" -key cert-key.pem -out cert.csr`

I also create a new file called `extfile.cnf` with the following content:

`subjectAltName=DNS:*.homelab.lan:192.168.0.17`

This specifies I want a wildcard cert that I can share across subdomains.

### Server Cert

After entering the encryption password for the CA's key, my cert.pem file is generated.

![server cert](/ssl/server_cert.png)

### Full Chain Cert

Now I combine the server cert together with the CA cert to create the full chain cert.

This is important so the client (my laptop) can receive both certs at once,

and know which CA had signed my server's cert so it can validate the signature.

I give it an easier name to recognize: `Homelab.pem`

![fullchain cert](/ssl/homelab_pem.png)

### Cleanup

I move my cert and private key into their corresponding dirs.

`sudo mv Homelab.pem /etc/ssl/certs`

`sudo mv cert-key.pem /etc/ssl/private`

## CA Trust

Last step is to add my CA's cert to my laptop's trusted list.

I sftp the ca.pem file to my laptop, 

then move it to `/etc/ca-certificates/trust-source/anchors/homelab-ca.crt`.

Running `sudo update-ca-trust extract` tells my system to reload the trusted list.

![trust CA cert](/ssl/add_ca_cert_to_trust.png)

Phew! Tedious, but complete.
