# Cryptographic Guidelines [Coding Practice]

<font size="-1">*Brandon Caldwell Jan 2019*</font>

## Overview

This document provides guidelines for how utilize encryption to protect data in transit and at rest. Cryptographic requirements at [COMPANY_NAME] are heavily influenced by NIST's Cryptographic Standards and Guidelines (CSRC) documentation.

- [Store the Cryptographic Hash of a Password](#store-the-cryptographic-hash-of-a-password)
- [Encrypt Sensitive Data At Rest](#encrypt-sensitive-data-at-rest)
- [Encrypt Data In Transit](#encrypt-data-in-transit)
- [Facilitate and Exercise Key Rotation](#facilitate-and-exercise-key-rotation)
- [Use Secure Random Number Generators](#use-secure-random-number-generators)
- [Store Private and Symmetric Keys in a Secure Location](#store-private-and-symmetric-keys-in-a-secure-location)

## Recommendations

### Store the Cryptographic Hash of a Password

User passwords should be stored as cryptographic hashes. Cleartext or encrypted password storage are not permitted. For more information on the reasons behind this, check out [this article](https://auth0.com/blog/hashing-passwords-one-way-road-to-security/) by Auth0. The Security Team recomends **using an algorithm like BCrypt, PBKDF2, or Argon2 **; these should be used with a decent work-factor. Hashed passwords should be salted with at least a 32-bit random salt. Remember that hashing algorithms are intentionally slow by design. The slower they are, the longer they take to crack.

##### Hash Algorithms for other user cases:

- Language Agnostic
  - SHA-512
  - SHA-384
  - SHA-256
- .Net hashing algoritms
  - SHA512Cng (FIPS compliant)
  - SHA384Cng (FIPS compliant)
  - SHA256Cng (FIPS compliant)
  - SHA512Managed (non-FIPS-compliant) *(use SHA512 as algorithm name in calls to `HashAlgorithm.Create` or `CryptoConfig.CreateFromName`)*
  - SHA384Managed (non-FIPS-compliant) *(use SHA384 as algorithm name in calls to `HashAlgorithm.Create` or `CryptoConfig.CreateFromName`)*
  - SHA256Managed (non-FIPS-compliant) *(use SHA256 as algorithm name in calls to `HashAlgorithm.Create` or `CryptoConfig.CreateFromName`)*
  - SHA512CryptoServiceProvider (FIPS compliant)
  - SHA256CryptoServiceProvider (FIPS compliant)
  - SHA384CryptoServiceProvider (FIPS compliant)

### Encrypt Sensitive Data At Rest

#### Symmetric Encryption Standard

Sensitive data should be encrypted when stored and then decrypted when accessed. Examples:

- Credentials intended to be used by the backend on behalf of the user (e.g., webhook credentials, VCS API tokens) 
- Personally Identifiable Information (PII)
- Medical Records
- Payment information

Maintained, trustworthy implementations of industry approved cryptographic algorithms, ciphers, and modes should be used to accomplish this. Writing your own cryptography is strongly discouraged.

**The Security Team recommends using AES-256 GCM as the algorithm and key-size for symmetric encryption**. Keep in mind that encryption standards are constantly changing, so it is important to keep your encryption and decryption functionality customizable and upgradable. If you are unsure if your data needs to be encrypted, consult with the [COMPANY_NAME] Governance and Compliance team or the [COMPANY_NAME] Legal team for help making that determination.

#### Authentication

Use a strong algorithm in AEAD (Authenticated Encryption with Associated Data) mode to encrypt sensitive data at rest. AEAD is a form of encryption that performs integrity checks on both the encrypted data and the contextual data associated with the cipher text. These checks protect encrypted data against chosen cipher text attacks. In order to prevent tampering of encrypted data, use AES-256 GCM (Galois/Counter Mode). GCM automatically performs authentication using the same key. If GCM is not used, use an HMAC (Hash-Based Message Authentication) to authenticate the AES encryption result.

A symmetric encryption flow should look roughly like:

AES -> HMAC -> Final Blob

#### Initialization-Vectors (IV) and Nonces

Most well known encryption libraries like Bouncy Castle and Conceal handle IV generation properly. If you need to manually generate an IV or nonce, make sure to use a Cryptographically Secure Pseudo-Random Number Generator (CSPRNG). Nonces should be at least 64 bits in length. The same IV or nonce should only be used once with a given symmetric key. See the "Secure Random Number Generation" recommendation below for a list of secure functions by language.

### Encrypt Data In Transit

#### Asymmetric Encryption Standard

Encryption should be used for any network communication between client-to-service and service-to-service communications. 

Maintained, trustworthy implementations of industry approved cryptographic algorithms, ciphers, and modes should be used to accomplish this. Writing your own cryptography is strongly discouraged.

RSA-4096 should be used for asymmetric encryption and store the private keys in a secure location outside of your codebase. If it is found that RSA-4096 becomes too resource intensive, it is acceptable to lower this to RSA-2048; this should only be a problem in very high-load environments. 

Enforce certificate verification in all HTTP clients. 

#### Usage Guidance

##### TLS - Cipher Suites

*Minimum TLS version = TLS 1.2.*

All cipher suites below should be compatible with version >= TLS 1.2. SSL is not allowed. TLS 1.0 and 1.1 are legacy protocols that lack modern security features like AEAD. Additionally, using TLS v1.0 violates PCI-DSS requirements. Modern browsers have deprecated TLS v1.0 and TLS v1.1 as of January 2020.

When organizing your server's list of ciphers, prioritize AEAD (GCM) modes and PFS (Perfect Forward Secrecy) (ECDHE) cipher suites.  

Accepted TLS Cipher suites:

- TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA
- TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
- TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
- TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
- TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
- TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
- TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
- TLS_DHE_RSA_WITH_AES_128_GCM_SHA256
- TLS_DHE_RSA_WITH_AES_256_GCM_SHA384
- TLS_DHE_RSA_WITH_AES_128_CBC_SHA
- TLS_DHE_RSA_WITH_AES_256_CBC_SHA
- TLS_DHE_RSA_WITH_AES_128_CBC_SHA256
- TLS_DHE_RSA_WITH_AES_256_CBC_SHA256

##### Recommended Libraries

- [libsodium](https://libsodium.org) - _([Github](https://github.com/jedisct1/libsodium))_
- [OpenSSL](https://www.openssl.org/) - _([Github](https://github.com/openssl/openssl))_

##### Resources

- https://github.com/ssllabs/research/wiki/SSL-and-TLS-Deployment-Best-Practices
- [Qualys SSL/TLS Server Test](https://www.ssllabs.com/ssltest/)

> NOTE: Where possible, we recommend Elliptic Curve Cryptography (ECC / ECDH), with Curve25519 (for signatures and key exchange)

### Facilitate and Exercise Key Rotation

All encryption keys have a lifetime. The longer they are actively used to encrypt data the higher the chance that the key will be leaked and data will be compromised. To protect against this, keys should be rotated at least every year and in highly sensitive environments like PCI it should occur every six months. 

In order to support periodic and on-demand (e.g., security incident) key rotation, your application should support reconfiguration of your encryption or hashing implementation. It should also facilitate m

When rotating keys it is often necessary for previous keys to stay around for a time to decrypt past data. A previous key should never be used to encrypt data once it is no longer the primary key. It should only be used to decrypt the data it was used to encrypt. When the data is ready to be stored again it should then be encrypted with the current key and stored.

### Use Secure Random Number Generators

#### Recommended Cryptographically Secure Pseudo-Random Number Generators (CSPRNG)

- **C:** getrandom(2)
- **CNG**: BCryptGenRandom(use of the BCRYPT*USE*SYSTEM*PREFERRED*RNG flag recommended unless the aller might run at any IRQL greater than 0 [that is, PASSIVE_LEVEL])
- **CAPI**: cryptGenRandom
- **Win32/64**: RtlGenRandom (new implementations should use BCryptGenRandom or CryptGenRandom)  *and_s*  SystemPrng (for kernel mode)
- **.NET:** RNGCryptoServiceProvider or RNGCng
- **PHP 5:** openssl_random_pseudo_bytes()
- **PHP 7:** random_bytes(), random_int()
- **Python:** secrets()
- **Ruby:** SecureRandom
- **Go:** crypto.rand
- **Rust:** rand::prng::chacha::ChaChaRng, CSPRNG libraries
- **Browser Javascript:** [Crypto.getRandomValues](https://developer.mozilla.org/en-US/docs/Web/API/Crypto/getRandomValues)
- **Windows Store Apps**: Windows.Security.Cryptography.CryptographicBuffer.GenerateRandom or GenerateRandomNumber
- **Objective-C (Apple OS X \*(10.7+)/iOS(2.0+))\***: int SecRandomCopyBytes (SecRandomRef random, size*t count, int8*t *bytes )
- **Apple OS X** *(<10.7)*: Use /dev/random to retrieve random numbers
- **Java** *(including Google Android Java code)*- java.security.SecureRandom class. Note that for Android 4.3 (Jelly Bean), developers must follow the Android recommended workaround and pdate their applications to explicitly initialize the PRNG with entropy from /dev/urandom or dev/random

Source: https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html#secure-random-number-generation

### Store Private and Symmetric Keys in a Secure Location

Private and symmetric keys should be stored in a secret storage solution that makes sense for your application. See our Secrets Management guidance for specific recommendations.

### References

- https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-52r1.pdf
- https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-52r2.pdf

### Contributors

* Jose Perez - Mar. 2020
* Andrew Luke - December 2021