---
linkTitle: DSR-RFC-02 Resource Access
title: DSR-RFC-02 Resource Access
weight: 2
---

Resource access can only be performed from the previously registered and fresh attested device.

## 1. High Level Flow

![ressource_access](resource_access.png)

1. `Policy Decision Point` regularly downloads, verifies, and installs the currently active policy from `Policy Administration Point` as well as context information from `Policy Information Point`.
2. `TrustClient` requests an attestation from the platform APIs. 
3. Attestation results are transmitted to `Device Management Service` in form of [Device Attestation Token](../dsr-rfc-04/#device-attestation-token-jwt_attest). Device Registration Service verifies the authenticity and integrity of the attestation and issues the [Device Token](../dsr-rfc-04/#device-token-device_token)
4. `TrustClient` connects to the `eHealth Service` using TLS. Mutual authentication is performed using the client certificate issued in [DSR-RFC-01]({{< ref "../dsr-rfc-01" >}})
5. `Trust Client`sends the `Device Token` as bearer token bound to mTLS certificate or a OAuth2 Code to the eHealth Service's `PEP`. PEP verifies the authenticity of the Device Token and extracts the device information.
6. `PEP` uses device information and other available signals (e.g. HTTP request headers) as input to the `PDP`. `PDP` applies the policy against the device information and any other input provided to it by the `PEP`.
7. Once `PDP` allowed the access by making the positive decision, the `PEP` lets the eHealth Service to continue and provide resources and other functionalities to the client.


## 2. Flow Details

{{% plantuml file="main_flow.puml" %}}


## 2.1 Android specifics

### 2.1.1 Create Android Device Attest

```plantuml
@startuml
autonumber "<b>['2.1.1' 00]"
skinparam defaultFontSize 10
skinparam DefaultMonospacedFontName Courier
skinparam lengthAdjust none

activate TrustClient

group Android
TrustClient -> TrustClient: trigger Play Integrity API(nonce_PlayIntegrityAPI),\n\tcreate keypair keypair_attest_derived(pubkey_mTLS),\n\tAttestCert_derived(nonce_attest_derived),\n\tcollect device_attributes_security
TrustClient -> TrustClient: create JWT_attest(\n\tTYPE_ANDROID,\n\tintegrity_verdict,\n\tAttestCert_derived,\n\tdevice_attributes_security,\n\tnonce)
TrustClient -> TrustClient: sign JWT_attest with keypair_attest_derived
end
```

### 2.1.2 Verify Android Device Attest

```plantuml
@startuml
autonumber "<b>['2.1.2' 00]"
skinparam defaultFontSize 10
skinparam DefaultMonospacedFontName Courier
skinparam lengthAdjust none

activate DMS

group Android
DMS -> DMS: extract\n\t""integrity_verdict""\n\t""AttestCert_derived""\n\t""device_attributes_security""
DMS -> DMS: verify\n\t""AttestCert_derived""\ncheck\n\t""AttestCert_derived is child""\n\t""of AttestCert_mTLS""
DMS -> GoogleServer ++: request\n\t""integrity_verdict(integrity_verdict)""
return return ""verdict_json""
DMS -> DMS: create\n\t""device_token(""\n\t""verdict_json,""\n\t""device_attributes_security,""\n\t""UUID_device <-> KVNR""\n\t"")""

end
```


## 2.2 Apple specifics

### 2.2.1 Create Apple Device Attest

```plantuml
@startuml
autonumber "<b>['2.2.1' 00]"
skinparam defaultFontSize 10
skinparam DefaultMonospacedFontName Courier
skinparam lengthAdjust none

activate TrustClient

group Apple
TrustClient -> TrustClient: trigger AppAttestAPI assertion(\n\tnonce_attest_derived | fingerprint(pubkey_mTLS)\n),\ncollect device_attributes_security
TrustClient -> TrustClient: create JWT_attest(\n\tTYPE_iOS,\n\tassertion,\n\tdevice_attributes_security,\n\tnonce)
TrustClient -> TrustClient: sign JWT_attest with keypair_attest_sign
end
```

### 2.2.2 Verify Apple Device Attest

```plantuml
@startuml
autonumber "<b>['2.2.2' 00]"
skinparam defaultFontSize 10
skinparam DefaultMonospacedFontName Courier
skinparam lengthAdjust none

activate DMS

group Apple
DMS -> DMS: extract assertion, device_attributes_security
DMS -> AppleCA ++ : verify assertion
return return result
group optional
  DMS -> AppleBackEnd ++: assess fraud risk
  return return assessment
end optional
DMS -> DMS: create device_token(\n\tassertion,\n\tdevice_attributes_security,\n\tUUID_device <-> KVNR\n)

end
```
