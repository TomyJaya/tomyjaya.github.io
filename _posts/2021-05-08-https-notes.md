---
layout:     post
title:      "HTTPS-related Concepts"
subtitle:   "Yet another notes"
date:       2021-05-08 12:00:00
author:     "Tomy Jaya"
header-img: "img/https-notes-bg.jpeg"
tags:
- https
---

Most developers know how HTTPS made browser to server communications more secure by encrypting the data in transport. Often times, though, we need to know more than just that. Here are some HTTPS-related concepts which I found useful in my work: 

## Does HTTPS use Asymmetric or Symmetric Encryption? 

This is a tricky question and sadly, commonly asked in interviews. The short answer is both! The PKI (Public Key Infrastructure) uses asymmetric encryption (public and private key) by design. It's great for distribution; nonetheless, it's uber slow. That's why, modern HTTPS uses both: 
   1) asymmetric encryption to establish a session by sharing a session key 
   2) the session key, used by both parties uses symmetric encryption to decrypt and encrypt corresponding data exchanged in a performant way. 

![https](https://i.stack.imgur.com/SGyYa.png)
**Source**: [infosecinstitute](https://resources.infosecinstitute.com/topic/ssl-dot-net-volume-1-hypothesis/)

## Public Keys vs Certificates

TLDR; a certificate contains a public key with some extra information to verify its validity/ authenticity. In the HTTPS world, a public key in itself is quite useless. We need to know who "signed" (aka certified) this public key and until when is this public key valid for. These public key "signers" are called CAs (Certificate Authorities). When a browser receives a certificate, it automatically checks its metadata about the CA who signed the public key. Browsers usually have an built-in list of trusted CAs to confirm the website domain is who they think they are. Once this is confirmed, we can use the website's public key to initiate a HTTPS session with assurance and the browser will usually display the lock icon. 

![certificate_vs_public_key.png](/img/certificate_vs_public_key.png)

## Can we do compression over HTTPS?

This is another tricky interview question. Logically, it seems like a good idea to save bandwidth. In practice, doing compression over HTTPS should be done selectively (i.e. with caution). Why? Because if the data exchanged is sensitive, TLS compression will make it vulnerable to length-related attacks like BREACH and CRIME. If performance is really your concern, here are some tips:
  - only compress static content like css, js, and public html (e.g. Terms and Conditions page)
  - use minification 
  - use browser cache headers

## Mutual TLS authentication (mTLS)

This concept is more common in a B2B or an Intranet setting. The idea is simple: HTTPS can also be used as a two-way authentication. On top of the traditional server authentication use-case, we can have client authenticate its identity by providing its certificate to the server. Provisioning the client certificates is operationally **not** very scalable, that's why it's only used in B2B integrations. In normal Internet web applications, client authentication is handled in the application layer. 

## Keystores vs keys

This was initially quite confusing for me, but it's actually very simple: you can load many keys (public or private) into a single keystore. The good thing with this approach is you can just have one password to secure and encrypt the keys in the keystore. Thus, you can load all the keys pertinent to say, one application, into one keystore and just have one password to guard them. 

## Keytore Types: `jks` vs `p12` 

JKS was a popular Java keystore format until Java 8. Nowadays it's deprecated. `.p12` (aka pkcs12) is the language agnostic alternative. PKCS12 file usually contains a bundled pair of private key and its corresponding certificate (incl. public key). But in theory, it can hold more than that.  

## What are self-signed certificates? 

Certificate Authorities (CAs) used to provide their service at a hefty fee. So, in the past, people employed workarounds to test their local webserver's SSL using self-signed certificates. This is nothing but having your public key digitally signed, not by a trusted CA, but by your own private key. I believe no modern browsers will accept self-signed certificates. The solution, nowadays, is usually either to have Intranet applications use internal company CAs or terminating SSL in the reverse proxy/ load balancer, such that application servers with business logic don't need to worry about SSL at all.  

## HMAC Using RSA

This might be quite a stretch from the main discussion about HTTPS, but since we're in the topic of web security and cryptography, let's revisit HMAC. We commonly use HMAC to as a message digest for secure file transfers, but it's actually a handy tool for API/ HTTP integrations as well. It complement HTTPS. The idea is, we use a shared key (e.g. MD5 or SHA-256) to sign our content string, and the receiver can do the exact thing to compare the signature and verify it comes from the trusted sender. We can go further. The key doesn't have to be a symmetric/ shared key! We can use asymmetric public & private key pair to do this too! That's where RSA comes in handy. We can use our private key to sign the message and the receiver can use our public key to verify if it indeed comes from us; all using the RSA algorithm. This reduces the hassle to have to communicate shared keys securely. AliPay employs this trick here: https://global.alipay.com/docs/ac/gr/globalgs

## Certificate Format, Encoding, & Extensions

Certificates have a standard format called `X.509`. This format is then most commonly encoded into 2 encodings:
  - PEM (Base64 ASCII) -> usually with `.pem` file extension
  - DER (Binary) -> usually with `.der` file extension
These file extensions are convention-based, I repeat, only convention-based. You should always check the content to see what it actually is. Other common extensions "conventions" include:
  - `.csr`: certificate signing request (the public key with metadata you send to the CA to sign)
  - `.crt`: people sometimes differentiate public key with certificates by having `.pem` extension reserved for public keys and `.crt` for certificates. 
  - `.key`: people usually use this extension to host private key. 


## Feedback welcome

Let me know if you have other HTTPS-related concepts you found useful, and I would be glad to add it to this list. 

## References: 
- https://superuser.com/questions/620121/what-is-the-difference-between-a-certificate-and-a-key-with-respect-to-ssl
- https://stackoverflow.com/questions/27039489/difference-between-signing-with-sha256-vs-signing-with-rsa-sha256/27041297
- https://stackoverflow.com/questions/2767211/can-you-use-gzip-over-ssl-and-connection-keep-alive-headers
- https://www.ssl.com/guide/pem-der-crt-and-cer-x-509-encodings-and-conversions/
