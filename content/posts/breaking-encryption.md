---
title: "Breaking Encryption" 
date: 2024-04-27T14:39:00.000Z 
categories: "articles" 
tags: 
  - "cryptography" 
params:
  math: true
draft: true 
---

<!-- toc -->

- [Introduction](#introduction)
  - [A Primer On Cryptography](#a-primer-on-cryptography)
- [A History Of Asymmetric Cryptography](#a-history-of-asymmetric-cryptography)
  - [Jumping Into The Details Of RSA](#jumping-into-the-details-of-rsa)
    - [RSA Key-Derivation Process](#rsa-key-derivation-process)
    - [Breaking an RSA Key](#breaking-an-rsa-key)

<!-- tocstop -->

# Introduction

Today encryption is all around us. Blockchains rely on it at their core, TLS
underpins today's web, pretty much all applications use some form of encryption
in one way or another, and if they don't - you probably shouldn't be using it.
Encryption is the backbone of modern communication. From secure online banking
to end-to-end messaging apps, encryption plays a vital role in keeping our data
safe.

In this article we are going to focus on: how encryption works and can be broken;
historic and current methods to do so on different forms of cryptosystems; and
finally a glimpse into the future of encryption.

## A Primer On Cryptography

As a quick primer we have on modern cryptography we have two main forms:

- **Symmetric cryptography**: where a single shared secret is used to both
  encrypt and decrypt messages. This key is shared between all parties involved
  and provides an efficient yet less secure form of encryption. It is important
  to keep this key from being leaked as unauthorised parties would have free
  reign over your encrypted data. So often it is delivered using the second
  form of encryption to ensure only the intended parties gain access to it.

- **Asymmetric cryptography**: where a public and private key pair are used to
  encrypt and decrypt messages respectively. Public keys are able to be shared
  and distributed freely - these are intended to be linked to someone/something
  and can be used to encrypt data for them. The private key is to be kept, as
  expected, private. It is the only thing capable of decrypting a message
  encrypted for its corresponding public key. As such key management is
  extremely important and should be done with caution and care.

Both methods have their benefits and drawbacks. symmetric cryptography is fast
but less secure, while asymmetric cryptography is more secure but generally slower.
As previously mentioned, these are often used in conjunction with one another.
Symmetric secret keys are encrypted for the recipient(s) using their public keys
and delivered securely, knowing only the intended recipient(s) can decrypt the
message and obtain the symmetric key. From then on the symmetric key will be
used for message encryption and decryption for, due to the speed advantage. The
combination of these two methods allows for efficient and secure end-to-end
encryption.

We will be focussing on asymmetric cryptosystems in this article, such as RSA,
Elliptic Curves and Lattices.

# A History Of Asymmetric Cryptography

Asymmetric cryptography is a relatively new form of cryptosystem, being developed
in the late 20th century. Whereas cryptography in general has existed for
thousands of years primarily in the form of symmetric cryptosystems such as the
famous Caesar cipher which uses a secret shift value to scramble the letters in
a message by shifting them alphabetically.

In the early 1970s cryptographers at GCHQ developed what would later become
RSA and the Diffe-Hellman key exchange. These findings were declassified by the
British government in 1997. However, in 1976 Whitfield Diffe and Martin Hellman
independently published publicly their asymmetric cryptosystem which came to be
known as the Diffe-Hellman key-exchange, which is still widely used today in
various forms. This allowed for the establishment of a shared secret without
any prior shared secret being used. This is done by applying public parameters
to each party's private key and exchanging the resulting public keys which are
equal to each other (a shared secret).

A year later in 1977, independent of GCHQ, MIT researchers Ron Rivest, Adi
Shamir and Leonard Adleman developed RSA and published it in 1978. RSA is an
asymmetric public-private key cryptosystem, based on the usage of large prime
numbers, to generate a pair of public and private keys through a series of steps.
RSA was a ground-breaking discovery in the field of cryptography and would be
the catalyst for the numerous innovations in the field that followed in the
years to come, such as Elliptic Curve based cryptography. First suggested in
1985 and later entering wide usage in 2004, Elliptic Curve cryptography allowed
for smaller keys compared to RSA with the same strength levels. This is done
by using elliptic curves over finite fields, a 256-bit elliptic curve public key
provides the same level of security as a 3072-bit RSA public key. This allows
for reduced storage and transmission requirements - improving the efficiency of
asymmetric cryptosystems in practice.

## Jumping Into The Details Of RSA

Let's dive into the details of how RSA key generation works, as it is relatively
simple and will give us an insight into our main point of discussion - how
encryption can be broken or cracked. As a cryptosystem is only as secure as the
best known method of breaking it - lots of research has been done to optimise
the attacking strategies. A cryptosystem once defined can only decrease in
security over time as attackers improve their strategies - unless the system is
changed.

### RSA Key-Derivation Process

1. First generate two secret large prime numbers $p$ and $q$.
1. Then calculate $n = p\cdot q$, this will be used as the key-size (in bits)
   and modulus going forward.
1. Calculate, using $\lambda$ (Carmichael's totient function) and the Euclidean
   algorithm:
   $$
   \begin{align*}
       \lambda(n)&= \text{lcm}(\lambda(p),\lambda(q)) \\
       &=\text{lcm}(\phi(p), \phi(q)) \\
       &=\text{lcm}(p-1,\ q-1) \\
       &=\frac{|(p-1)(q-1)|}{\text{gcd}(p-1,\ q-1)}
   \end{align*}
   $$
   This works as $n=p\cdot q$ so $\lambda(n)=\text{lcm}(\lambda(p),\ \lambda(q))$.
   Due to both $p$ and $q$ being prime $\lambda(p)=\phi(p)=p-1$. Finally as
   $a\cdot b=\text{gcd}(a,\ b)\cdot\text{lcm}(a,\ b)$ we can use the Euclidean
   algorithm to quickly compute this secret value.
1. Then we compute the public key exponent $e$ by choosing an integer such that
   $1\<e\<\lambda(n)$ and $\text{gcd}(e,\ \lambda(n))=1 $, effectively choosing an
   integer that is coprime to $\lambda(n)$. The smaller $e$ is the less secure
   the key can be, so $e$ is usually chosen as $2^{16}-1$.
1. The public key is then essentially $(e,\ n)$, the pairing of the public
   exponent and modulus. As prime factorisation is computationally expensive it
   is deemed secure to publish $n$, but we will come back to this.
1. The private key exponent $d$ is the modular multiplicative inverse of
   $e \mod\lambda(n)$
   $$
   \begin{alignat}{2}
       d&\equiv e^{-1} &\mod\lambda(n) \\
       d\cdot e&\equiv 1 &\mod\lambda(n)
   \end{alignat}
   $$
   As $e$ and $\lambda(n)$ are coprime, $d$ can be computed using the extended
   Euclidean algorithm. This is because $e\cdot d+\lambda(n)\cdot x=\text{gcd}
   (e,\ \lambda(n))=1$ where $x$ is the modular multiplicative inverse of
   $\lambda(n) \mod e$. Thus the extended Euclidean algorithm is efficient to
   compute $d$.
1. This leaves the private key to be $(d,\ p,\ q,\ \lambda(n))$ or
   simply $(d)$ as the other secret fields can be omitted since the key
   derivation process is complete to save space.

### Breaking an RSA Key

To finally define the term "breaking", I mean it as taking public data and using
it to derive the private key for a specific public key. This would allow you to
to not only decrypt messages encrypted for the corresponding public key, but
also encrypt messages impersonating the original private key's owner. This is
an extremely dangerous possibility, but a fun problem to attempt to solve.

As the public key of an RSA key-pair is $(e,\ n)$ and the entire key-derivation
process originates by calculating $n=p\cdot q$. If you were able to prime
factorise $n$ you would be able to derive $p$ and $q$, as each integer has a
unique prime factorisation. This sounds simple but is rather complex the larger
$p$ and $q$ become. Simple Brute Force attacks are too complex to use, so we
must look into more advanced prime factorisation techniques.

{{< subscribe-button >}}
