---
title: "SSL/TLS Handshake, Hybrid Encryption and Public Key Infrastructure (PKI)"
date: 2022-01-22T14:00:33+02:00
categories:
- tech
tags:
- security
- encryption
---

### Hybrid Encryption: Symmetric and Asymmetric Encryption Combined

Both symmetric and asymmetric encryption has advantages and disadvantages. So, which one should we use? Well, nowadays
we often use them together. Asymmetric encryption is often used to exchange private keys between parties securely. In
other words, parties who would communicate establish an asymmetric encryption protocol in the beginning just to
exchange private keys. When the private key exchange is completed, they keep communicating by using symmetric
encryption - which is faster than asymmetric encryption.  This is also how SSL/TLS works.

Using symmetric and asymmetric encryption together is also known as _Hybrid Encryption_.

### Hybrid Encryption for Communications: Messaging

Here is a sample scenario for hybrid communications:

- Bob wants to send an e-mail to Alice.
- Bob gets Alice's digital certificate, which includes her name and her public key.
  - Alice's certificate must be signed by a CA (certificate authority). In other words, it must be encrypted with a
    CA's private key.
- Bob decrypts Alice's certificate, by using CA's public key, and reveals her public key.
- Bob generates a symmetric secret key pair (two identical keys).
- Bob uses Alice's public key to encrypt one of these symmetric keys.
- The only key that can decrypt this message and reveal the symmetric key is Alice's private key.
- Therefore, Bob is safe to send this key over the wire, and he sends it to Alice.
- Alice then uses her private key to decrypt the message and to reveal the symmetric key of Bob.
- From now on, they both can use this symmetric key to encrypt and decrypt messages between them.
- Briefly, asymmetric encryption is used to facilitate a key exchange.

> This is exactly how SSL/TLS works!

### SSL/TLS Handshake

> The content under this title is identical with the steps explained under "Hybrid Encryption for Communications: Messaging"
> Alice here replaced by google.com

- Bob wants to connect to google.com.
- Bob gets google.com's digital certificate, which includes the organization name and their public key.
  - google.com's certificate must be signed by a CA (certificate authority). In other words, it must be encrypted with
    a CA's private key.
- Bob decrypts google.com's certificate, by using CA's public key, and reveals their public key.
- Bob generates a symmetric secret key pair (two identical keys).
- Bob uses google.com's public key to encrypt one of these symmetric keys.
- The only key that can decrypt this message and reveal the symmetric key is google.com's private key.
- Therefore, Bob is safe to send this key over the wire, and he sends it to google.com.
- google.com then uses their private key to decrypt the message and to reveal the symmetric key of Bob.
- From now on, they both can use this symmetric key to encrypt and decrypt messages between them.
- Briefly, asymmetric encryption is used to facilitate a key exchange.

### SSL or TLS?

Often SSL and TLS are used interchangeably. However, they aren't identical terms. SSL (Secure Sockets Layer) is a
deprecated cryptographic protocol and totally by TLS (Transport Layer Security). The last time SSL was updated was in
1996.

SSL and TLS have three main purposes:
  - Confidentiality (Encryption): Data is only accessible by Client and Server.
  - Integrity (Hashing): Data isn't modified between Client and Server.
  - Authentication (PKI): Client/Server is indeed who they say they are.

#### TLS Downgrade Attack

TLS is prone to "Downgrade Attack". In this attack type, the client (web browser) fakes the highest supported TLS
version with an old version. For example, the client requests TLS 1.0, even though it can support TLS 1.3. By doing so,
the attacker tries to be served by a weaker TLS version and manipulate it. To prevent your systems from this attack,
you should configure your server not to support lower TLS versions.

### Who are these CAs (Certificate Authorities) anyway?

- Today, 5 organizations (SSL certificate issuers) secure 98% of the Internet:
  - IdenTrust (51.9%)
    - Let's Encrypt
  - DigiCert (19.4%)
    - GeoTrust
    - Verisign
    - Thawte
  - Sectigo (17.5%)
    - Comodo
  - GoDaddy (6.9%)
  - GlobalSign (2.9%)

However, they don't issue all certificates themselves. Often we obtain our certificates from intermediate CAs, which
is a topic of PKI (Public Key Infrastructure).

## Public Key Infrastructure (PKI)

CA's use self-signed certificates! But why? Because we need to trust someone, at the top! At the top, there is a CA,
which has a self-signed certificate, such as IdenTrust. Then there are intermediate CAs, under the root CA, but their
certificates aren't self-signed. Instead, they are signed by a CA (with CA's private key). These intermediate CAs can
issue (sign) certificates for us. This is known as the "Chain of Trust". We trust our certificate provider, our
certificate provider trusts the root CA, and so on. By the way, it's also possible to set up your own corporate CA if
you don't want to pay for certificates every single time.

In other to verify the _Chain of Trust_, we use their public keys. Let's say google.com has a certificate issued by CA1.
To verify this, we check the certificate of CA1. The thing is, the certificate of CA1 is issued by RootCA. Now we
need to verify everything one by one. Our browser will verify the RootCA by checking the local trust store. Then the
certificate of CA1 by using the public key of RootCA. Then the certificate of google.com, by using the public key of CA1.

Three entities form a PKI:

- Client: Needs to connect securely or verify identity.
- Server: Needs to prove its identity.
- Certificate Authority: Validate identities & generate certificates.
