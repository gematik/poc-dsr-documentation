---
linkTitle: DSR-RFC-01 Device Registration
title: DSR-RFC-01 Device Registration
weight: 1
---

Device registration is performed by the user after previous authentication.

## 1. High Level Flow

![registration_overview](device_registration.png)

1. At first start, the [`TrustClient`]({{< relref "/docs/concepts/trust-client/">}}) generates the [app/device identity key material]({{< relref "/docs/concepts/trust-client/_index.md#app--device-identity" >}}) and a corresponding [`Certificate Signing Request`](../dsr-rfc-04/#trust-client-certificate-signing-request-crl).
2. The [`TrustClient`]({{< relref "/docs/concepts/trust-client/" >}}) then triggers the [attestation of the key material, device and app]({{< relref "/docs/concepts/trust-client/_index.md#device-and-app-attestation" >}}) with the help of the `Platform Attestation Service` and receives attestation results.
3. To register the device for themself, the user signs a [Device Registration Token](../dsr-rfc-04/#device-registration-token-jwt_registration), containing the [app/device identity and attestation results]({{< relref "/docs/concepts/trust-client/_index.md#app--device-identity" >}}) and the [`Certificate Signing Request`](../dsr-rfc-04/#trust-client-certificate-signing-request-crl) with their electronic health card (`EF.C.CH.AUT.E256`). Then the [`TrustClient`]({{< relref "/docs/concepts/trust-client/" >}}) sends the [Device Registration Token](../dsr-rfc-04/#device-registration-token-jwt_registration) to the `Device Management Service`.
4. With the help of the `Platform Attestation Service`, the `Device Management Service` verifies the received attestations and key material.
5. The `Device Management Service` sends the [`Certificate Signing Request`](../dsr-rfc-04/#trust-client-certificate-signing-request-crl) to the `DMS CA` which then issues a [`Trust Client Certificate`](../dsr-rfc-04/#trust-client-certificate-signing-request-crl)
6. Finally, the `Device Management Service` stores the [app/device identity key material]({{< relref "/docs/concepts/trust-client/_index.md#app--device-identity" >}}) with the corresponding user identity in its database.
7. The `Device Management Service` sents the [`Trust Client Certificate`](../dsr-rfc-04/#trust-client-certificate-signing-request-crl) to the [`TrustClient`]({{< relref "/docs/concepts/trust-client/">}}).

## 2. Flow Details

{{% plantuml file="main_flow.puml" %}}

## 2.1 Android specifics

### 2.1.1 Create Android Device Registration

```plantuml
@startuml
autonumber "<b>['2.1.1' 00]"
skinparam defaultFontSize 10
skinparam DefaultMonospacedFontName Courier
skinparam lengthAdjust none

activate TrustClient
participant PlayIntegrityAPI


group Android
TrustClient -> TrustClient: generate\n\t""keypair_attest""\n\t""attestCertChain(nonce_attest)""\n\t""keypair_mTLS""\n\t""keypair_mTLS_cert(nonce_keypair_mTLS)""\n\t""CSR(nonce_CSR_mTLS, keypair_mTLS)""
TrustClient -> PlayIntegrityAPI ++: perform IntegrityTokenRequest(nonce_Integrity)
return return integrityVerdict

TrustClient -> TrustClient: create\n\t""JWT_registration(""\n\t\t""TYPE_ANDROID,""\n\t\t""nonce,""\n\t\t""pubkey_mTLS,""\n\t\t""keypair_mTLS_cert,""\n\t\t""pubkey_attest,""\n\t\t""attestCertChain""\n\t\t""integrityVerdict,""\n\t\t""CSR""\n\t"")""

end

@enduml
```


### 2.1.2 Verify Android Device Registration

```plantuml
@startuml
autonumber "<b>['2.1.1' 00]"
skinparam defaultFontSize 10
skinparam DefaultMonospacedFontName Courier
skinparam lengthAdjust none

activate DMS
participant GoogleServer


group Android
DMS -> DMS: extract nonce, pubkey_mTLS, AttestCert_mTLS,\nintegrityVerdict, CSR from JWS_registration_signed
DMS -> DMS: verify attestCertChain and certification extension data
DMS -> DMS: verify AttestCert_mTLS and certification extension data
DMS -> DMS: verify AttestCert_mTLS was signed by pubkey_attest
DMS -> GoogleServer ++: get Key Attestation certificate revocation status list

return return CRL_json
DMS -> DMS: evaluate CRL_json
DMS -> GoogleServer ++: request integrity_verdict(integrityVerdict)
return return verdict_json
DMS -> DMS: evaluate verdict_json

end

@enduml
```

## 2.2 Apple specifics

### 2.2.1 Create Apple Device Registration

```plantuml
@startuml
autonumber "<b>['2.2.1' 00]"
skinparam defaultFontSize 10
skinparam DefaultMonospacedFontName Courier
skinparam lengthAdjust none

activate TrustClient
participant AppAttestAPI

group Apple
TrustClient -> TrustClient: generate\n\t""keypair_mTLS""\n\t""CSR(nonce_CSR_mTLS, keypair_mTLS)""
TrustClient -> AppAttestAPI ++: generate\n\t""keypair_attest(""\n\t\t""SHA256(nonce_Integrity | SHA256(pubkey_mTLS))""\n\t"")""
AppAttestAPI -> AppAttestAPI: attest

return return attestation_statement

TrustClient -> TrustClient: create\n\t""JWT_registration(""\n\t\t""TYPE_IOS,"",\n\t\t""nonce,""\n\t\t""pubkey_mTLS,""\n\t\t""attestation_statement,""\n\t\t""keyIdentifier_attest,""\n\t\t""CSR""\n\t"")""

end

@enduml
```


{{% codeblock language="java" file="../apple-samples/Sources/DeviceSecurityRating/TrustService.swift" start=59 end=128 %}}


### 2.2.2 Verify Apple Device Registration

```plantuml
@startuml
autonumber "<b>['2.2.2' 00]"
skinparam defaultFontSize 10
skinparam DefaultMonospacedFontName Courier
skinparam lengthAdjust none

activate DMS


group Apple
DMS -> DMS: extract nonce, pubkey_mTLS, attestation_statement,\nkeyIdentifier_attest, CSR
DMS -> DMS: verify attestation_statement
end

@enduml
```
