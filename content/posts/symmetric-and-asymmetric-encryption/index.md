---
title: "Symmetric and Asymmetric Encryption"
description: "A detailed scenario-based comparison of symmetric and asymmetric encryption methods."
date: 2022-01-21T14:00:33+02:00
categories:
- tech
tags:
- security
- encryption
cover:
    image: "posts/symmetric-and-asymmetric-encryption/assets/rue-a-saint-parize-le-chatel-pres-de-nevers-1862-johan-barthold-jongkind.jpg"
    alt: "Rue A Saint-Parize-Le-Chatel, Pres De Nevers (1862) - Johan Barthold Jongkind"
    relative: false
images: ["assets/rue-a-saint-parize-le-chatel-pres-de-nevers-1862-johan-barthold-jongkind.jpg"]
---

### Symmetric Encryption (Private Key Cryptography)

In symmetric encryption only a single key, in other words, a private key is used to encrypt and decrypt a message.
Symmetric encryption is also known as "Private Key Cryptography" as the whole encryption is only based on a private key.

Some popular _symetric_ encryption algorithms are:

| Algorithm | Cipher | Key Size              | Block/State Size | Popularity | Notes                                  |
|-----------|--------|-----------------------|------------------|------------|----------------------------------------|
| AES       | Block  | 128, 192, or 256 bits | 128 bits         | 1          | The best one. Still in use.            |
| Blowfish  | Block  | 32-448 bits           | 64 bits          | 2          | Still quite safe, but slower than AES. |
| Twofish   | Block  | 128, 192, or 256 bits | 128 bits         | 3          | Still quite safe, but slower than AES. |
| DES       | Block  | 56 bits               | 64 bits          | 4          | Not considered as safe anymore.        |
| 3DES      | Block  | 112, or 168 bits      | 64 bits          | 4          | Not considered as safe anymore.        |
| RC4       | Stream | 40-2048 bits          | 2064 bits        | 4          | Not considered as safe anymore.        |

**Pros of symmetric encryption:**

- It's fast. Much faster than asymmetric encryption.
- Ciphertext is the same size as plain text. There is no cipher-text expansion.

**Cons of symmetric encryption:**

- Securely exchanging the private key on any medium is the biggest challenge as the key can be stolen during the
  exchange.
- Every person with a copy of a private key is now a security risk because they can make the key stolen.
- Rotating the private key is challenging as all copies of the private key also need to be rotated.

#### Block and Stream Ciphers

Any symmetric encryption algorithm either uses a block or a stream cipher.

| Block Ciphers                               | Stream Ciphers                   |
|---------------------------------------------|----------------------------------|
| Used to encrypt chunks or blocks of data    | Used to encrypt streams of data  |
| Good for known sizes of data                | Better for unknown sizes of data |
| More popular than stream ciphers            | Not commonly used these days     |
| Examples: AES, Blowfish, Twofish, DES, 3DES | Examples: RC4                    |

Stream ciphers are less secure than block ciphers as a malicious hacker can flip a bit while the algorithm is running
an XOR bit-by-bit. Flipping a single bit generates a whole different message and wreck the output in block ciphers as
they are encrypted altogether as a block.

However, stream ciphers can be protected with HMAC (Hashed Message Authentication Code).

### Asymmetric Encryption (Public Key Cryptography)

- Asymmetric encryption is also known as _Public Key Cryptography_.
- In asymmetric encryption, two different keys (public key and private key) are used for encryption and decryption.
- As the name suggests, the public key can be distributed publicly over the wire. However,  the private key must be
  protected.
- These two keys (public key and private key) are mathematically related pairs, but still, they aren't identical copies.
- In asymmetric encryption, the public key is used to encrypt a message, and a private key is used to decrypt a message.
- However, asymmetric encryption algorithms can also be used for _signing_ purposes. In this case, the private key is used
  to _sign_ a message, and a public key is used to verify a signature.
- Keep in mind, encryption and signing serve different purposes! Encryption is all about _confidentiality_, and signing
  is all about _integrity_ and _authentication_.
- Asymmetric encryption algorithms in general, depends on huge prime number calculations that are impossible to break if
  the key size is sufficient.

Some popular _asymmetric_ encryption algorithms are:

| Algorithm      | Purpose/Usage                        |
|----------------|--------------------------------------|
| RSA            | Encryption, signatures, key exchange |
| DSA            | Signatures                           |
| Diffie-Hellman | Key exchange                         |
| ElGamal        | Encryption, signatures               |

**Pros of asymmetric encryption:**

- No key exchange concerns. The public key can be shared publicly over the wire.

**Cons of asymmetric encryption:**

- Compared to symmetric encryption, it's very slow and not efficient for large data.
- _Cipher Text Expansion_: The ciphertext is larger than the original message.

#### Asymmetric Encryption and Man in the Middle

In theory, the public key can be shared publicly over the Internet, and anyone can use this key to encrypt their
messages and send them back to us. However, how can we be sure of the public key that we are using for encryption? Since
anyone can create a key pair and publish it on the Internet, this model is open to impersonation. What if an attacker
tricks you with their public key? Since you will be encrypting your message with the attacker's public key, they will
be able to read all messages with their private key. They can even do that without being noticed. This is how it works:

- An attacker replaces a legit public key with their public key.
- The victim encrypts their message with the attacker's public key and sends the encrypted message.
- The attacker receives the message, decrypts it with their private key, and then encrypts it with the legit key again
  and sends the encrypted message to the legit key owner.
- By doing so, the attacker can remain undetected between the communication.

So, how do we deal with this Man in the Middle attack? The short answer is _signing_ and _certificates_.

### Digital Signatures

Digital signatures are all about Integrity, Authentication, and Non-Repudiation. However, they don't provide
confidentiality. Therefore you can think of hashing algorithms as they are the default way of proving integrity
when you hear about digital signatures.

Anything can be signed digitally. An e-mail, a file, a message - anything you can think of.

Whatever you're signing should go through a hashing algorithm to generate a digest. When a digest is encrypted with the
sender's private key it's called a _digital signature_. The receiver uses the sender's public key to decrypt the message
and to reveal the digest. Then the receiver generates the hash again and compares it with what they got after decryption.

### Digital Certificates

One of the problems with asymmetric encryption is, we can never be sure if we are using the real public key of the
recipient. Therefore, an independent and trusted authority, similar to a notary, that binds a public key with a name is
required. When a name with a public key is bound, it's called a _digital certificate_.

- A digital certificate includes the following:
  - Owner's name
  - Owner's public key and its expiration date
  - The certificate issuer's name
  - The certificate issuer's digital signature

A digital certificate is based on trust (chain of trust), to the issuer. When using digital certificates, the trusted
the third party is a certificate authority, an entity that issues digital certificates.

## Sample Scenarios

### Asymmetric Encryption for Confidentiality: Without Digital Certificates

> Receiver's public key is used for encryption
> This is how GPG/PGP encryption process works.

- Bob wants to send an e-mail to Alice.
- Bob uses Alice's public key to encrypt his message.
- Alice receives the e-mail and decrypts it with her private key.

|                                | Guaranteed? | Why?                                                                                    |
|--------------------------------|-------------|-----------------------------------------------------------------------------------------|
| Confidentiality                | Yes         | The message is encrypted. Only Alice, with her private key, can decrypt it              |
| Integrity                      | Yes         | Alice can decrypt the message. That proves the message is encrypted with her public key |
| Non-repudiation/Authentication | No          | Anyone can get Alice's public key, encrypt a message with it, and send it to her        |

Man-in-the-middle scenario:

- The attacker intercepts the communication and receives the message from Bob.
- The attacker can't decrypt the message as they will need the private key of Alice.
- The attacker can't also modify the message because if they do, Alice won't be able to decrypt it with her private message.

Risks:

- If an attacker can replace Alice's public key, or somehow trick Bob with their public key, then they can control the
  whole communication without being noticed.
- Alice can never be sure of the sender, as anyone can send her a message by using her certification.

### Asymmetric Encryption for Confidentiality: With Digital Certificates

> Receiver's public key is used for encryption

- Bob wants to send an e-mail to Alice.
- Bob fetches Alice's certificate, obtains her public key, and checks the "certificate owner's name" to make sure it
  belongs to Alice.
- Bob uses Alice's public key to encrypt his message.
- Alice receives the e-mail and decrypts it with her private key.

|                                | Guaranteed? | Why?                                                                                    |
|--------------------------------|-------------|-----------------------------------------------------------------------------------------|
| Confidentiality                | Yes         | The message is encrypted. Only Alice, with her private key, can decrypt it              |
| Integrity                      | Yes         | Alice can decrypt the message. That proves the message is encrypted with her public key |
| Non-repudiation/Authentication | No          | Anyone can get Alice's certificate, encrypt a message with it, and send it to her       |

Man-in-the-middle scenario:

- The attacker intercepts the communication and receives the message from Bob.
- The attacker can't decrypt the message as they will need the private key of Alice.
- The attacker can't also modify the message because if they do, Alice won't be able to decrypt it with her private message.

Risks:

- Alice can never be sure of the sender, as anyone can send her a message by using her certification.

### Asymmetric Encryption for Integrity and Authentication: Without Digital Signatures

> Sender's private key is used for encryption

- Bob wants to send an e-mail to Alice.
- Bob uses his private key to encrypt the message.
- Alice receives the e-mail and decrypts it with Bob's public key.

|                                | Guaranteed? | Why?                                                                                                                |
|--------------------------------|-------------|---------------------------------------------------------------------------------------------------------------------|
| Confidentiality                | No          | Anyone can decrypt the encrypted message since Bob's public key is publicly available                               |
| Integrity                      | Yes         | Alice can decrypt the message. That proves, the message is encrypted with Bob's private key and hasn't been changed |
| Non-repudiation/Authentication | Yes         | Since the message can be decrypted with Bob's public key, it must have been encrypted with his private key          |

Man-in-the-middle scenario:

- The attacker intercepts the communication and receives the message from Bob.
- The attacker can decrypt the message as the public key of Bob is available to everyone.
- The attacker can't modify the message because if they do, Bob's public key won't work during decryption.

The problem with this approach is, the asymmetric encryption is slow and it doesn't work well with big messages
because of the _cipher text expansion_ (the ciphertext is larger than the original message). Therefore, we often use
digital signatures for integrity and authentication purposes.

### Asymmetric Encryption for Integrity and Authentication: With Digital Signatures

> Sender's private key is used for encryption
> This is how GPG/PGP signing process works.

- Bob wants to send an e-mail to Alice.
- Bob hashes the plain-text message and generates a digest.
  - The generated digest is smaller than the original message, so it saves us from the previously explained
    _cipher text expansion_ issue.
- Bob uses his private key to encrypt the digest.
  - This encrypted digest is called as _digital signature_.
- Bob sends the message in plaintext and the digital signature to Alice.
- Alice receives the e-mail and decrypts the digital signature with Bob's public key to reveal the digest.
- Alice hashes the plaintext message and generates her digest. Then she compares the digest she created, and she
  got from Bob. If they match, integrity and non-repudiation will be proven.

|                                | Guaranteed? | Why?                                                                                                                                   |
|--------------------------------|-------------|----------------------------------------------------------------------------------------------------------------------------------------|
| Confidentiality                | No          | The message is in plain text, and digital signatures aren't about confidentiality                                                      |
| Integrity                      | Yes         | Alice can decrypt the message and compare hashes. That proves, the message is encrypted with Bob's private key and hasn't been changed |
| Non-repudiation/Authentication | Yes         | Since the message can be decrypted with Bob's public key, it must have been encrypted with his private key                             |

Man-in-the-middle scenario:

- The attacker intercepts the communication and receives the message from Bob.
- The attacker can decrypt the message and reveal the digest as the public key of Bob is available to everyone.
- The attacker can't modify the message because if they do, Bob's public key won't work during decryption.

### Asymmetric Encryption for Integrity and Authentication: With Digital Certificates

> Sender's private key is used for encryption

- Bob wants to send an e-mail to Alice.
- Bob obtains a certificate by using his public key.
- Bob uses his private key to encrypt the message.
- Alice uses Bob's certificate to obtain his public key and to verify his name.
- Alice decrypts the e-mail with Bob's public key.

|                                | Guaranteed? | Why?                                                                                                                |
|--------------------------------|-------------|---------------------------------------------------------------------------------------------------------------------|
| Confidentiality                | No          | Anyone can decrypt the encrypted message since Bob's certificate is publicly available                              |
| Integrity                      | Yes         | Alice can decrypt the message. That proves, the message is encrypted with Bob's private key and hasn't been changed |
| Non-repudiation/Authentication | Yes         | Since the message can be decrypted with Bob's certificate, it must have been encrypted with his private key         |

Man-in-the-middle scenario:

- The attacker intercepts the communication and receives the message from Bob.
- The attacker can decrypt the message as the certificate of Bob is available to everyone.
- The attacker can't modify the message because if they do, Bob's certificate won't work during decryption.

Risks:

- No confidentiality as Bob's certificate is available to everyone.

### Asymmetric Encryption for Confidentiality, Integrity and Authentication: With Digital Certificates

> Sender's private key is used for encryption

- Bob wants to send an e-mail to Alice.
- Bob fetches Alice's certificate, obtains her public key, and checks the "certificate owner's name" to make sure it
  belongs to Alice.
- Bob encrypts his message with his private key.
- Bob re-encrypts the encrypted message with Alice's public key.
- Bob send the message to Alice.
- Alice gets the message and decrypts it with her private key to reveal the message that Bob encrypted with his
  private key.
- Alice then decrypts it with Bob's public key.

|                                | Guaranteed? | Why?                                                                                                                                   |
|--------------------------------|-------------|----------------------------------------------------------------------------------------------------------------------------------------|
| Confidentiality                | Yes         | The message is encrypted with Alice's public key. Only Alice, with her private key, can decrypt it                                     |
| Integrity                      | Yes         | Alice can decrypt the message and compare hashes. That proves, the message is encrypted with Bob's private key and hasn't been changed |
| Non-repudiation/Authentication | Yes         | Since the message can be decrypted with Bob's public key, it must have been encrypted with his private key                             |

The problem with this approach is, the asymmetric encryption is slow and it doesn't work well with big messages
because of the _cipher text expansion_ (the ciphertext is larger than the original message). Since Bob uses asymmetric
encryption for encrypting his message, the process will be slow and the output might be huge. Therefore, we often use
digital signatures for integrity and authentication purposes.

Man-in-the-middle scenario:

- The attacker intercepts the communication and receives the message from Bob.
- The attacker can't decrypt the message as they can't obtain Alice's private key.
- The attacker can't modify the message because if they do, Bob's public key won't work during decryption.
- The sender is guaranteed to be Bob as he signed the message with his private key, a key that attackers can't obtain.

### Asymmetric Encryption for Confidentiality, Integrity and Authentication: With Digital Certificates and Signatures

> Sender's private key is used for integrity
> Reciever's public key is used for encryption

- Bob wants to send an e-mail to Alice.
- Bob fetches Alice's certificate, obtains her public key, and checks the "certificate owner's name" to make sure it
  belongs to Alice.
- Bob hashes the plain-text message and generates a digest.
  - The generated digest is smaller than the original message, so it saves us from the previously explained
    _cipher text expansion_ issue.
- Bob uses his private key to encrypt the digest.
  - This encrypted digest is called as _digital signature_.
- Bob encrypts his message with Alice's public key.
- Bob sends the encrypted message and the digital signature to Alice.
- Alice receives the e-mail and
  - Decrypts the digital signature with Bob's public key to reveal the digest.
  - Decrypts the encrypted message with her private key.
- Alice hashes the decrypted message and generates her digest. Then she compares the digest she created, and she
  got from Bob. If they match, integrity and non-repudiation will be proven.

|                                | Guaranteed? | Why?                                                                                                                                   |
|--------------------------------|-------------|----------------------------------------------------------------------------------------------------------------------------------------|
| Confidentiality                | Yes         | The message is encrypted with Alice's public key. Only Alice, with her private key, can decrypt it                                     |
| Integrity                      | Yes         | Alice can decrypt the message and compare hashes. That proves, the message is encrypted with Bob's private key and hasn't been changed |
| Non-repudiation/Authentication | Yes         | Since the message can be decrypted with Bob's public key, it must have been encrypted with his private key                             |

Man-in-the-middle scenario:

- The attacker intercepts the communication and receives the message from Bob.
- The attacker can't decrypt the message as they can't obtain Alice's private key.
- The attacker can't modify the message because if they do, Bob's public key won't work during decryption.
- The sender is guaranteed to be Bob as he signed the message with his private key, a key that attackers can't obtain.

### Summary and Comparison

|                                                                                                                   | Confidentiality | Integrity | Authentication | Performance | Man in the Middle |
|-------------------------------------------------------------------------------------------------------------------|-----------------|-----------|----------------|-------------|-------------------|
| Asymmetric Encryption for Confidentiality: Without Digital Certificates                                           | Yes             | Yes       | No             | Fast        | Vulnerable        |
| Asymmetric Encryption for Confidentiality: With Digital Certificates                                              | Yes             | Yes       | No             | Fast        | Safe              |
| Asymmetric Encryption for Integrity and Authentication: Without Digital Signatures                                | No              | Yes       | Yes            | Slow        | Vulnerable        |
| Asymmetric Encryption for Integrity and Authentication: With Digital Signatures                                   | No              | Yes       | Yes            | Fast        | Vulnerable        |
| Asymmetric Encryption for Integrity and Authentication: With Digital Certificates                                 | No              | Yes       | Yes            | Slow        | Vulnerable        |
| Asymmetric Encryption for Confidentiality, Integrity and Authentication: With Digital Certificates                | Yes             | Yes       | Yes            | Slow        | Safe              |
| Asymmetric Encryption for Confidentiality, Integrity and Authentication: With Digital Certificates and Signatures | Yes             | Yes       | Yes            | Fast        | Safe              |
