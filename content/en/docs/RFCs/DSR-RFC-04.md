---
linkTitle: DSR-RFC-04
title: DSR-RFC-04 Security Tokens
weight: 4
---

{{% pageinfo %}}
Content is under development
{{% /pageinfo %}}

## Device Registration Tokens

```json
// header
{
  "typ": "JWT",
  "alg": "BP256R1", // gematik custom alg!!
  "x5c": [
    "<EF.C.CH.AUT.E256>",
    "<intermediates_n>" //see RFC-7515, 4.1.6 String | Base64 encode DER Cert
  ]
} 
 
// body (Android)
{
  "iss": "TrustSDK_<SDKVersion>",
  "sub": "<pubkey_mTLS_fingerprint (SHA256)>",
  "iat": <NumericDate>,
  "type": "android",
  "nonce": "<String>", // base64 encoded bytes
  "csr": "<csr>", // base64 encoded PKCS#10
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