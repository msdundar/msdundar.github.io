---
title: "Storing Passwords in a Database: Hashing, Salts, and Peppers"
date: 2022-01-24T01:30:33+02:00
categories:
- tech
tags:
- security
- hashing
---

_"How do you store passwords in a database"_? A very common question for back-end-oriented interviews. After conducting
hundreds of technical interviews on different levels, I can confidently say around 50% of the candidates can't answer
this question. The most common answer I often got is
_"there is a package/gem/library we use, and it manages the password part_".

Well, yes, frameworks, libraries, and packages cover most of the complexity nowadays, but I don't accept this as an
excuse for not being curious about essentials. Frameworks, packages, libraries, tools, text editors - they come and go
all the time, but essentials just don't. The ways we use to store passwords haven't been changed much since I started
programming, which was 15 years ago.

## Encryption or Hashing?

Encryption and hashing are different concepts, however, they're often confused with each other. Even seasoned engineers
use these terms interchangeably from time to time. Let's take a look at how they're different:

|                | Encryption                                                       | Hashing                                                                         |
|----------------|------------------------------------------------------------------|---------------------------------------------------------------------------------|
| Is about       | Confidentiality                                                  | Integrity                                                                       |
| Reversibility  | Yes. An encrypted message can also be decrypted.                 | No. Hashing is one-way. A hashed message can't be converted back to plain text. |
| Preservability | Yes. Encryption encodes and preserves 100% of the original text. | No. Hashing produces a fixed-length signature.                                  |
| Collisions     | No. Encryption always produces a unique output.                  | Yes. Hashing may (in rare cases) produce hash collisions.                       |

When storing passwords in a database what we need isn't encryption, as we don't want these passwords to be decrypted.
Instead, what we need is hashing, to check the integrity of a user's account when they want to log in again. However,
a database as a whole can be encrypted (usually with
[asymmetric encryption](https://www.mysql.com/products/enterprise/encryption.html)), but that's a different topic.

Let's explore some common techniques when storing passwords.

## Hashing

Passwords stored in a database must be hashed with a strong hashing algorithm. Nowadays, MD5, SHA-0, and SHA-1 aren't
secure enough, therefore you should never use these algorithms anymore.

SHA-512/224, SHA-512/256, and SHA3 are considered safe, considering collision attacks and length extension attacks,
but they have a common problem, and they aren't good for hashing passwords. Because they're fast!

Isn't speed something good? A fast algorithm sounds pretty good at first glance, right? Well, the thing is, it's not
only fast for you, but also for hackers who wish to create a rainbow table or run a brute-force attack as fast as
possible. Therefore, you don't always want fast algorithms, sometimes slowness is what you seek.

With a fast algorithm, an attacker can compute billions of hashes per second and create a huge rainbow table quickly.
However, a slow algorithm will make things extra challenging, and sometimes impossible for them.

[bcrypt](https://en.wikipedia.org/wiki/Bcrypt) and [scrypt](https://en.wikipedia.org/wiki/Scrypt) has specifically
designed for password-hashing and to be slow for example. The number of iterations during hashing can be configured if
you ever need to make the algorithm slower. Nowadays, [Argon2](https://en.wikipedia.org/wiki/Argon2) is also getting a
lot of attention for some it's the successor of bcrypt already. There is also a group of people who prefer many
iterations of SHA-512 over bcrypt, so the hashing algorithm is a very controversial topic. If you aren't sure about
which password hashing algorithm to use, you can always refer to
[OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html).

Briefly, do your own research, and pick a strong password hashing algorithm.

## Salts

A salt is a unique random number that is added to each password before hashing.

  ```
  hash("password" + "RANDOM_SALT_STRING")
  ```

- The salt doesn't need to be private. It can be stored in clear text in the database. The only purpose of a salt is to
  defeat rainbow tables.
- Salting also prevents two users from having the same password hash, even if they use the same password.
- A salt is typically stored in the database together with the hashed password.

Salting prevents an attacker from:
  - Recognizing known hashes (rainbow tables etc.)
  - Cracking one password may crack many (if users use the same password)
  - Seeing if two users have the same password

But doesn't stop:
  - Brute forcing a single password (an attacker can still generate hashes as salt will also be leaked with passwords)

Modern hashing algorithms such as Argon2id, bcrypt, scrypt, and PBKDF2 automatically salt the passwords, so no
additional steps are required when using them.

## Peppers

There are two types of password peppers.

1. a randomly selected pepper for each user
2. a shared secret pepper.

### Randomly Selected Pepper for Each User

This technique is very rare but still mentioned in a couple of resources.

  ```
  hash("password" + "RANDOM_SALT_STRING" + random("a", "z"))
  ```

- A pepper is a short string or character appended to the end of a password.
- Peppers are random and different for each password.
- The website cycles through all possible peppers (a-z for example) until a matching hash is found, during login. If
  a combination matches with the entered password, then access is granted to the user.
- The pepper isn't stored in the database! No one knows what the pepper is, not even the website. It's usually defined
  as a range in the code.
- Using a pepper increases time to brute force, multiplied by the number of possible peppers. Likewise, using a pepper
  also increases the time to log in for a user, multiplied by the number of possible peppers. But this is usually not a
  problem.

### A Shared Secret Pepper

A more common technique adopted as default by many popular frameworks nowadays.

  ```
  hash("password" + "RANDOM_SALT_STRING" + ENV["SECRET_PEPPER"])
  ```

- A pepper is a fixed random string, that is added to the password + salt combination.
- Compared to password salt a pepper isn't unique for each password, instead, it's a fixed value.
- A pepper isn't stored in the database! Typically it's an environment variable in the server, or hardcoded in the
  application code. Therefore, an attacker can't do much by only leaking the database. They also need to hack the
  server.

Cheers.
