# Cryptographic Guidelines [Coding Practice]
<font size="-1">*Brandon Caldwell Jan 2019*</font>

## Guidelines Overview

Cryptographic requirements at Unity are heavily influenced by NIST's *Cryptographic Standards and Guidelines (CSRC)* documentation. Below are our condensed recommendations.
### Recommended Libraries
- [libsodium](https://libsodium.org) - _([Github](https://github.com/jedisct1/libsodium))_
- [OpenSSL](https://www.openssl.org/) - _([Github](https://github.com/openssl/openssl))_

> NOTE: Where possible, we recommend Elliptic Curve Cryptography (ECC / ECDH), with Curve25519 (for signatures and key exchange)

### Usage Recommendations
##### TLS - Cipher Suites

_Minimum TLS version = TLS1.2._

All cipher suites below should be compatible with version >= TLS 1.2 . SSL 3.0 and below are not allowed. 

Accepted TLS Cipher suites:

- TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_ECDSA_WITH_AES_128_CCM
- TLS_ECDHE_ECDSA_WITH_AES_256_CCM
- TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
- TLS_AES_128_GCM_SHA256
- TLS_AES_256_GCM_SHA384
- TLS_AES_128_CCM_SHA256

##### Hash Algorithms

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

###### Random Number Generators (RNG)
- **CNG**: BCryptGenRandom(use of the BCRYPT_USE_SYSTEM_PREFERRED_RNG flag recommended unless the aller might run at any IRQL greater than 0 [that is, PASSIVE_LEVEL])
- **CAPI**: cryptGenRandom
- **Win32/64**: RtlGenRandom (new implementations should use BCryptGenRandom or CryptGenRandom) * and_s * SystemPrng (for kernel mode)
- **.NET** RNGCryptoServiceProvider or RNGCng
- **Windows Store Apps**: Windows.Security.Cryptography.CryptographicBuffer.GenerateRandom or GenerateRandomNumber
- **Apple OS X** _(10.7+)/iOS(2.0+)_: int SecRandomCopyBytes (SecRandomRef random, size_t count, int8_t *bytes)
- **Apple OS X** _(<10.7)_: Use /dev/random to retrieve random numbers
- **Java** _(including Google Android Java code)_- java.security.SecureRandom class. Note that for Android 4.3 (Jelly Bean), developers must follow the Android recommended workaround and pdate their applications to explicitly initialize the PRNG with entropy from /dev/urandom or dev/random

### References
- https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-52r1.pdf
- https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-52r2.pdf

### Contributors

Jose Perez - Mar. 2020