---
linkTitle: DSR-RFC-04 Security Tokens
title: DSR-RFC-04 Security Tokens
weight: 4
---

{{% pageinfo %}}
Content is under development
{{% /pageinfo %}}

## Introduction
tbd

## Device Registration Token (JWT_registration)

* sent from TrustClient to GMS during registration
* contains all needed information for registration
* signed with EF.C.CH.AUT.E256 (eGK)
* see [DSR-RFC-01]({{< ref "/docs/rfcs/dsr-rfc-01" >}} "DSR-RFC-01") for corresponding sequence diagram

```json
// header
{
  "typ": "JWT",
  "alg": "BP256R1",
  "x5c": [
    "<EF.C.CH.AUT.E256>",
    "<intermediates_n>" //see RFC-7515, 4.1.6 String | Base64 encode DER Cert
  ]
} 
 
// body (Android)
{
  "iss": "TrustSDK_<SDKVersion>",
  "sub": "<pubkey_mTLS_fingerprint (SHA256)>",
  "subjectCert": "<cert>" (attested by attestKeyPair) // base64 encoded
  "iat": <NumericDate>,
  "type": "android",
  "nonce": "<String>", // base64 encoded bytes
  "csr": "<csr>", // base64 encoded PKCS#10
  "attestPulicKey": "<pubkey_attest_fingerprint (SHA256)>",
  "attestCertChain": [
    "<leaf cert>",
    "<intermediate cert>",
    "<...>"
  ], // base64 encoded der certs
  "integrityVerdict": "<Sring>", //encrypted integrity verdict exact from google response
  "packageName": "..."
}  
 
// body (Apple)
{
  "iss": "TrustSDK_<SDKVersion>", // String
  "sub": "<pubkey_mTLS_fingerprint (SHA256)>", // String | Base64 encoded sha256
  "iat": <NumericDate>, // int
  "type": "apple", // String
  "nonce": "<nonce>", // String | Base64 encoded binary data
  "csr": "<csr>", // String | Base64 encoded PKCS#10
  "keyId": "<keyIdentifier>", //String | Base64 encoded binary data - for attestation required key identifier
  "attestation": "<attestationObject>" // String | Base64 encoded CBOR
}    
 
// signature
{..}
```

## Device Attestation Token (JWT_attest)

* sent from TrustClient to GMS for attestation
* contains attestation information
* signed with mTLS_privateKey, (Android), privateKey_attest (iOS)
* see [DSR-RFC-02]({{< ref "/docs/rfcs/dsr-rfc-02" >}} "DSR-RFC-02") for corresponding sequence diagram

```json
// header
{
  "typ": "JWT",
  "alg": "ES256", // NIST P-256,
  "x5c": [
    "<cert_mTLS>",
    "<intermediate_1>",
    "<intermediate_n>" //see RFC-7515, 4.1.6>
  ]
}
 
// body (Android)
{
  "iss": "TrustSDK_<SDKVersion>", // String
  "sub": "<String>", // Base64 encoded sha256 pubkey_mTLS_fingerprint (SHA256)
  "iat": <NumericDate>,
  "type": "android",   "nonce": "<String>", // base64 encoded bytes
  "attestationCertChain": [
    "<attestation_cert>",
    "<attest_cert_1>",
    "<attest_cert_n>",
    "<..>"
  ], // base64 encoded der certs (fresh generated attestationCert + attestCertChain)
  "integrityVerdict": "<String>", //encrypted integrity verdict exact from google response
  "packageName": "<String>",
  "deviceAttributes": { //see DSR-RFC-06
    "build": {
      "version": {
        "sdkInit": "<int>", // Build.VERSION.SDK_INIT,
        "securityPatch": "<String>" // Build.VERSION.SECURITY_PATCH
      },
      "manufacturer": "<String>",
      "product": "<String>",
      "model": "<String>",
      "board": "<String>"
    },
    "ro": {
      "crypto": {
        "state": "<boolean>" // ro.crypto.state
      },
      "product": {
        "firstAPILevel": "<int>" //ro.product.first_api_level
      }
    },
    "packageManager": {
      "featureVerifiedBoot": "<boolean>" // PackageManager.FEATURE_VERIFIED_BOOT
      "mainLinePatchLevel": "<String>"
    },
    "keyguardManager": {
      "isDeviceSecure": "<boolean>" // KeyguardManager.isDeviceSecure()
    },
    "biometricManager": {
      "deviceCredential": "<boolean>", // BiometricManager.canAuthenticate(BiometricManager.Authenticators.DEVICE_CREDENTIAL) or BiometricManager.canAuthenticate(Authenticators#DEVICE_CREDENTIAL)
      "biometricStrong": "<boolean>" // BiometricManager.canAuthenticate(Authenticators#BIOMETRIC_STRONG)
    },
    "devicePolicyManager": {
      "passwordComplexity": "<int>" //DevicePolicyManager.getPasswordComplexity(), PASSWORD_COMPLEXITY_NONE, PASSWORD_COMPLEXITY_LOW, PASSWORD_COMPLEXITY_MEDIUM, or PASSWORD_COMPLEXITY_HIGH
    }
  }
}  
 
// body (Apple)
{
  "iss": "<String>", // TrustSDK_<SDKVersion>
  "sub": "<String>", // Base64 encoded pubkey_mTLS_fingerprint (SHA256)>
  "iat": <NumericDate>,
  "type": "apple",
  "nonce": "<String>", // base64 encoded bytes
  "assertion": "<CBOR>", //CBOR assertionObject
  "deviceAttributes": { //see DSR-RFC-06
    "UIDevice": {
      "systemName": "<String>", // UIDevice.systemName
      "systemVersion": "<String>", // UIDevice.systemVersion
      "identifierForVendor": "<UUID>" // UIDevice.identifierForVendor
    }
    "appVersion" : "<String>",
  }
}  
 
// Signature
{..}
```

## Device Token (device_token)

* sent from GSM to TrustClient to PEP (or directly from GSM to PEP)
* contains standard values from OAuth2.0 Certificate-Bound Access Tokens and
    * verdict_json ( Google Integrity API)
    * assertion (iOS)
    * device_attributes_security (see [dsr-rfc-06]({{< ref "/docs/rfcs/dsr-rfc-06" >}} "DSR-RFC-06"))
    * UUID_device ↔ KVNR relation
* signed with privateKey_GMS
* see [DSR-RFC-02]({{< ref "/docs/rfcs/dsr-rfc-02" >}} "DSR-RFC-02") for corresponding sequence diagram

```json
// Header
{
 "typ": "JWT",
 "alg": "ES256", // NIST P-256,
 "x5c": [
    "<cert_GMS>",
    "<intermediate_1>",
    "<intermediate_n>"
  ] // see RFC-7515, 4.1.6>
}
 
// body (Android)  {
  "iss": "<String>", // GMS-ID or Domain
  "sub": "<String>", // pubkey_mTLS_fingerprint (SHA256)
  "iat": "<NumericDate>",
  "exp": "<int>", // expiration date
  "jti": "<UUID>", // random UUID - for blacklisting before expiration
  "type": "android",
  "userIdentifier": "<String>", // KVNR
  "deviceHealth": { //see DSR-RFC-06
    "integrityVerdict": {
      "appIntegrity": {
        "appRecognitionVerdict": "<String>", // PLAY_RECOGNIZED, UNRECOGNIZED_VERSION, UNEVALUATED
        "packageName": "<String>", // e.g. com.package.name
        "certificateSha256Digest": "<String>", // e.g. 6a6a1474b5cbbb2b1aa57e0bc3
        "versionCode": "<String>" // e.g. 42
      },
      "deviceIntegrity": {
        "deviceRecognitionVerdict": [
          "<String>",
          "<String>"
        ] //  MEETS_DEVICE_INTEGRITY, MEETS_BASIC_INTEGRITY, MEETS_STRONG_INTEGRITY, MEETS_VIRTUAL_INTEGRITY, empty string
      },
      "accountDetails": {
        "appLicensingVerdict": "<String>" // LICENSED, UNLICENSED, or UNEVALUATED
      }
    },
    "keyIdAttestation": {
      "attestationVersion": "<int>", // 1, 2, 3, 4, 100, 200
      "attestationSecurityLevel": "<int>", // Software  (0),TrustedEnvironment  (1), StrongBox  (2),
      "keyStore": {
        "type": "<String>", // KEY_MASTER, KEY_MINT
        "version": "<int>",
        "securityLevel": "<int>" // Software  (0),TrustedEnvironment  (1), StrongBox  (2),
      },
      // attestation challenge is not forwarded!
      "softwareEnforced": { // content is optional
        // see DSR-RFC-06
      },
      "teeEnforced": { // content is optional
        // see DSR-RFC-06
      }
    },
    "deviceAttributes": { //see DSR-RFC-06
      "build": {
        "version": {
          "sdkInit": "<int>", // Build.VERSION.SDK_INIT,
          "securityPatch": "<String>" // Build.VERSION.SECURITY_PATCH
        },
        "manufacturer": "<String>",
        "product": "<String>",
        "model": "<String>",
        "board": "<String>"
      },
      "ro": {
        "crypto": {
          "state": "<boolean>" // ro.crypto.state
        },
        "product": {
          "firstAPILevel": "<int>" //ro.product.first_api_level
        }
      },
      "packageManager": {
        "featureVerifiedBoot": "<boolean>" // PackageManager.FEATURE_VERIFIED_BOOT
      },
      "keyguardManager": {
        "isDeviceSecure": "<boolean / null>" // KeyguardManager.isDeviceSecure()
      },
      "biometricManager": {
        "deviceCredential": "<boolean / null>", // BiometricManager.canAuthenticate(BiometricManager.Authenticators.DEVICE_CREDENTIAL) or BiometricManager.canAuthenticate(Authenticators#DEVICE_CREDENTIAL)
        "biometricStrong": "<boolean / null>" // BiometricManager.canAuthenticate(Authenticators#BIOMETRIC_STRONG)
      },
      "devicePolicyManager": {
        "passwordComplexity": "<int>" //DevicePolicyManager.getPasswordComplexity(), PASSWORD_COMPLEXITY_NONE, PASSWORD_COMPLEXITY_LOW, PASSWORD_COMPLEXITY_MEDIUM, or PASSWORD_COMPLEXITY_HIGH
      }
    }
  },
  "cnf": { // confirmation claim, RFC-8705
    "x5t#S256": "bwcK0esc3ACC3DB2Y5_lESsXE8o9ltc05O89jdN-dg2" //  X.509 Certificate SHA-256 Thumbprint
  }
}  
 
// body (Apple)  
{
  "iss": "<String>", // GMS-ID or Domain
  "sub": "<String>", // pubkey_mTLS_fingerprint (SHA256)
  "iat": "<NumericDate>",
  "exp": "<int>", // expiration date
  "jti": "<UUID>", // random UUID - for blacklisting before expiration
  "type": "apple",
  "userIdentifier": "<String>", // KVNR
  "deviceHealth": { // see DSR-RFC-06
    "assertion": {
      "rpID": "<int>", // A hash of your app’s App ID, which is the concatenation of your 10-digit team identifier, a period, and your app’s CFBundleIdentifier value.
      "counter": "<int>", // A value that reports the number of times your app has used the attested key to sign an assertion.
      "riskMetric": "<int>" //  indicates the number of attested keys associated with a given device over the lifetime of the device. Look for this value to be a low number.
    },
    "deviceAttributes": { //see DSR-RFC-06
      "UIDevice": {
        "systemName": "<String>", // UIDevice.systemName
        "systemVersion": "<String>", // UIDevice.systemVersion
        "identifierForVendor": "<UUID>" // UIDevice.identifierForVendor
      },
      "appVersion": "<String>"
    }
  },
  "cnf": { // confirmation claim, RFC-8705
    "x5t#S256": "bwcK0esc3ACC3DB2Y5_lESsXE8o9ltc05O89jdN-dg2" // X.509 Certificate SHA-256 Thumbprint
  }
}  
 
// Signature
{..}
```

## Trust Client Certificate Signing Request (CRL)
* sent from TrustClient to GMS and then to GMS CA for requesting / issuing Cert_mTLS
* contains standardized information
* signed by TrustClient with privKey_mTLS
* see [DSR-RFC-01]({{< ref "/docs/rfcs/dsr-rfc-01" >}} "DSR-RFC-01") for corresponding sequence diagram

```text
// PKCS #10
Common Name = TRUST_CLIENT
Country Name = DE
Organization = DSR_POC
```

## Trust Client Certificate (cert_mTLS)

* issued by GMS CA to TrustClient / device
* attests the device's/app's pubkey_mTLS (device identity)
* must contain KVNR 
* see [DSR-RFC-01]({{< ref "/docs/rfcs/dsr-rfc-01" >}} "DSR-RFC-01") for corresponding sequence diagram

```json
//tbd
```

## Session Token (session_token_fd)
* sent from health service (FD) to TrustClient after successful device rating
* allows using session for a defined period of time without another device attestation
* signed by health service (FD)
* see [DSR-RFC-02]({{< ref "/docs/rfcs/dsr-rfc-02" >}} "DSR-RFC-02") for corresponding sequence diagram
```json
// header
{
  "typ": "JWT",
  "alg": "ES256", // NIST P-256,
  "x5c": [
    "<cert_FD>",
    "<intermediate_1>",
    "<intermediate_n>"
  ], //see RFC-7515, 4.1.6> // OR
  "jwk": { // example!
    "kty": "EC",
    "crv": "P-256",
    "x": "f83OJ3D2xF1Bg8vub9tLe1gHMzV76e8Tus9uPHvRVEU",
    "y": "x_FEzRu9m36HLN_tue659LNpXW6pCyStikYjKIWI5a0",
    "kid": "Public key used in JWS spec Appendix A.3 example"
  }
}
 
// body
{
  "iss": "https://server.example.com", //Fachdienst FQDN
  "sub": "brian@example.com", // Fachdienst internal user ID
  "iat": 1467324320, // issuance date
  "exp": 1467324920, // expiration date
  "jti": "<unique token ID>", // for blacklisting before expiration
  "cnf": { // confirmation claim, RFC-8705
    "x5t#S256": "bwcK0esc3ACC3DB2Y5_lESsXE8o9ltc05O89jdN-dg2" //  X.509 Certificate SHA-256 Thumbprint
  }
}
 
// signature
{..}
```