---
title: "SSL/TLS Handshake, Hybrid Encryption and Public Key Infrastructure (PKI)"
description: "Hybrid encryption, SSL/TLS handshake and Public Key Infrastructure (PKI) explained."
date: 2022-01-22T14:00:33+02:00
categories:
- tech
tags:
- security
- encryption
- ssl
- tls
cover:
    image: "posts/ssl-tls-handshake-hybrid-encryption/assets/paysage-a-la-cote-saint-andre-1886-johan-barthold-jongkind.jpg"
    alt: "Paysage a la Côte Saint-Andre (1886) - Johan Barthold Jongkind"
    relative: false
images: ["assets/paysage-a-la-cote-saint-andre-1886-johan-barthold-jongkind.jpg"]
---

### Hybrid Encryption: Symmetric and Asymmetric Encryption Combined

Both symmetric and asymmetric encryption has advantages and disadvantages. So, which one should we use? Well, nowadays
we often use them together. Asymmetric encryption is often used to exchange private keys between parties securely. In
other words, parties who would communicate establish an asymmetric encryption protocol in the beginning just to
exchange private keys. When the private key exchange is completed, they keep communicating by using symmetric
encryption - which is faster than asymmetric encryption.  This is also how SSL/TLS works.

Using symmetric and asymmetric encryption together is also known as _Hybrid Encryption_.

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

> This is exactly how SSL/TLS also works!

### SSL/TLS Handshake

> The content under this title is identical with the steps explained under "Hybrid Encryption for Communications: Messaging"
> Alice here replaced by google.com. Some additional handshake-specific steps were added to the list.

- Bob wants to connect to google.com.
- Bob's browser sends a `ClientHello` message to the server including:
  - Supported highest TLS version
  - Supported cryptographic algorithms
- At this point, the server knows what the client can do. The server picks one of the supported TLS versions,
  cryptographic algorithms, and then replies to the client with a `ServerHello` message including:
  - The chosen TLS version
  - The chosen cryptographic algorithm
  - Session ID
  - Server's digital certificate: includes the organization name and their public key. This certificate must have been
    signed by a CA (certificate authority). In other words, it must have been encrypted with a CA's private key.
- Bob decrypts google.com's certificate, by using CA's public key, to check certificate validity.
- The server sends a `ServerKeyExchange` message to the client. This message includes:
  - Parameters for the key exchange.
  - A digital signature (hash) of a set of previous messages so far, signed with the private key of the server. It's
    proof that the server is who they say they are.
- Bob checks the validity of this digital signature.
- Bob generates a symmetric secret key pair (two identical keys).
- Bob sends a `ClientKeyExchange` message to the server and initiates the key exchange process.
- Bob uses google.com's public key to encrypt one of these symmetric keys. The only key that can decrypt this message
  and reveal the symmetric key is google.com's private key. Therefore, Bob is safe to send this key over the wire, and
  he sends it to google.com.
- Bob sends a `ClientFinished` message, including digital signature (hash) of all previous messages so far.
- google.com then uses their private key to decrypt the message and to reveal the symmetric key of Bob.
- google.com sends a `ServerFinished` message, including the digital signature (hash) of all previous messages so far.
  By doing so, both parties can be sure that no one intercepted the communication (man-in-the-middle).
- From now on, they both can use this symmetric key to encrypt and decrypt messages between them. Briefly, asymmetric
  encryption is used to facilitate a key exchange.

  ```
          Client                                               Server

          ClientHello
          (empty SessionTicket extension)-------->
                                                          ServerHello
                                      (empty SessionTicket extension)
                                                          Certificate*
                                                    ServerKeyExchange*
                                                  CertificateRequest*
                                        <--------      ServerHelloDone
          Certificate*
          ClientKeyExchange
          CertificateVerify*
          [ChangeCipherSpec]
          Finished                     -------->
                                                      NewSessionTicket
                                                    [ChangeCipherSpec]
                                        <--------             Finished
          Application Data             <------->     Application Data
  ```

### SSL or TLS?

Often SSL and TLS are used interchangeably. However, they aren't identical terms. SSL (Secure Sockets Layer) is a
deprecated cryptographic protocol and totally by TLS (Transport Layer Security). The last time SSL was updated was in
1996.

SSL and TLS have three main purposes:
  - Confidentiality (Encryption): Data is only accessible by Client and Server.
  - Integrity (Hashing): Data isn't modified between Client and Server.
  - Authentication (PKI): Client/Server is indeed who they say they are.

#### TLS Downgrade Attack

Old versions of TLS are prone to "Downgrade Attack". In this attack type, the client (web browser) fakes the highest
supported TLS version with an old version. For example, the client requests TLS 1.0, even though it can support TLS 1.3.
By doing so, the attacker tries to be served by a weaker TLS version and manipulate it. To prevent your systems from
this attack, you should configure your server not to support lower TLS versions. In the latest version of TLS, the
`ClientFinished` and `ServerFinished` messages are added to eliminate these downgrade attacks.

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
