---
linkTitle: DSR-RFC-09 Health Service
title: DSR-RFC-09 Health Service API
weight: 9
---

{{% pageinfo %}}
Content is under development
{{% /pageinfo %}}

## Structure of the Health Service

A concrete instance of a FD service is identified via FQDN 

| Application / Service  | Provider  | FQDN  |
|---|---|---|
| DSR "dummy" health service  | tbd  | dsr.health-service.example.com |

The DSR "dummy" health service provides the following business APIs / resources

| API | Version  | URL  |
|---|---|---|
| emergency data API | v1  | https://dsr.health-service.example.com/api/v1/notfalldaten |
| ePrescription API | v1  | https://dsr.health-service.example.com/api/v1/erezept/<id> |
| ePrescription API | v2  | https://dsr.health-service.example.com/api/v2/erezept/<id>  |

Scaling and load balancing is done in the background, invisible to the clients.

### API Documentation

see [poc-dsr-fd](https://github.com/gematik/poc-dsr-fd/blob/main/src/main/resources/openapi.yaml) on GitHub.

### Test Data ePrescription API

| Version  | ID  | 
|---|---|
| v1  | 6c371ed4-f83a-47b4-8249-446852a5b382 |
| v1  | 17e5970d-6e64-465f-be0c-ed0f5d7624a0 |
| v2  | 00fa65b9-2e07-40c7-b8c5-fadf549c4ed7 |
| v2  | 01e18e40-83a5-4986-819f-c68c2227b59d |

### Test Data emergency data API

| patientIdentifier  | 
|---|
| X123456 |
| X234567 |
| X345678 |

## Input Data for Policy Decision Point

Input data for access decisions by the `Policy Decision Point` is independent of the DSR "dummy" health Service or its APIs and results from the external sources. The `Policy Enforcement Point` is responsible to determine appropriate input, verify and submit to the `Policy Decision Point` for decision according to a JSON schema:

### Input Example for iOS Devices

```json
{
  "request": {
    "method": "GET",
    "path": "/api/v1/notfalldaten",
    "clientIP": "XXX.XXX.XXX.XXX",
    "countryCode": "DE"
  },
  "deviceTokenValid": true,
  "deviceTokenPayload": {
    "type": "IOS",
    "userIdentifier": "X114428530",
    "deviceHealth": {
      "assertion": {integrityVerdict         "counter": 1,
        "rpID": "XXXXXX",
        "riskMetric": "unavailable"
      },
      "deviceAttributes": {
        "systemVersion": "16.6",
        "systemName": "iOS",
        "identifierForVendor": "XXXXXX",
        "systemModel": "iPhone13,3"
      }
    },
    "iss": "DSR GMS 1.0.0",
    "sub": "XXXX",
    "iat": 1234567890,
    "exp": 1234567890,
    "cnf": {
      "x5t#S256": "XXXXXX"
    },
    "jti": "XXXXXX"
  }
}
```

### Input Example for Android Devices

```json
{
  "request": {
    "method": "GET",
    "path": "/api/v1/notfalldaten",
    "clientIP": "XXX.XXX.XXX.XXX",
    "countryCode": "DE"
  },
  "deviceTokenValid": true,
  "deviceTokenPayload": {
    "type": "ANDROID",
    "userIdentifier": "TEST KVNR",
    "deviceHealth": {
      "integrityVerdict": {
        "appIntegrity": {
          "appRecognitionVerdict": "UNRECOGNIZED_VERSION",
          "packageName": "de.gematik.dsr.android",
          "certificateSha256Digest": [
            "XXXXXX"
          ],
          "versionCode": 1
        },
        "deviceIntegrity": {
          "deviceRecognitionVerdict": [
            "MEETS_DEVICE_INTEGRITY"
          ]
        },
        "accountDetails": {
          "appLicensingVerdict": "UNLICENSED"
        }
      },
      "keyIdAttestation": {
        "attestationVersion": 200,
        "attestationSecurityLevel": 1,
        "softwareEnforced": {
          "creationDateTime": "2023-07-24T10:59:02.933Z",
          "usageExpireDateTime": "2024-07-23T12:59:02Z",
          "activeDateTime": "2023-07-24T09:59:02Z",
          "attestationApplicationId": {
            "packageInfos": [
              {
                "packageName": "de.gematik.dsr.android",
                "version": 1
              }
            ],
            "signatureDigests": [
              "XXXXXX"
            ]
          },
          "rootOfTrust": null,
          "originationExpireDateTime": "2024-07-23T12:59:02Z"
        },
        "teeEnforced": {
          "purpose": [
            2,
            3
          ],
          "attestationIdDevice": "XXXXXX",
          "keySize": 256,
          "osVersion": 130000,
          "origin": 0,
          "osPatchLevel": 202306,
          "attestationIdModel": "XXXXXX",
          "attestationIdProduct": "XXXXXX",
          "vendorPatchLevel": 20230605,
          "attestationApplicationId": null,
          "noAuthRequired": true,
          "rootOfTrust": {
            "verifiedBootKey": "XXXXXX",
            "deviceLocked": true,
            "verifiedBootState": "VERIFIED",
            "verifiedBootHash": "XXXXXX"
          },
          "algorithm": 3,
          "digest": [
            4
          ],
          "ecCurve": 1,
          "attestationIdManufacturer": "XXXXXX",
          "attestationIdBrand": "XXXXXX",
          "bootPatchLevel": 20230605
        },
        "keyStore": {
          "type": "KEY_MINT",
          "version": 200,
          "securityLevel": 1
        }
      },
      "deviceAttributes": {
        "build": {
          "version": {
            "sdkInit": 33,
            "securityPatch": "2023-06-05"
          },
          "manufacturer": "Google",
          "product": "oriole",
          "model": "Pixel 6",
          "board": "oriole"
        },
        "ro": {
          "crypto": {
            "state": true
          },
          "product": {
            "firstAPILevel": 31
          }
        },
        "packageManager": {
          "featureVerifiedBoot": true
        },
        "keyguardManager": {
          "isDeviceSecure": true
        },
        "biometricManager": {
          "deviceCredential": true,
          "biometricStrong": true
        },
        "devicePolicyManager": {
          "passwordComplexity": 327680
        }
      }
    },
    "iss": "DSR GMS 1.0.0",
    "sub": "XXXXXX",
    "iat": 1234567890,
    "exp": 1234567890,
    "cnf": {
      "x5t#S256": "XXXXXX"
    },
    "jti": "XXXXXX"
  }
}
```

The structural check of the ``device token`` (signature, validity, mTLS fingerprint matching) is performed in the ``Policy Enforcement Point`` before the ```Policy Decision Point``` is called. The result of this check is passed to the ```Policy Decision Point``` with the `deviceTokenValid` attribute.

## Access Decision / Verdict by the PDP

For example, the positive decision look like this:

```json
{
  "allow": true,
  "device": {
    "allow": true,
    "violations": []
  },
  "security": {
    "allow": true,
    "violations": []
  }
}
```

A negative decision provides hints about which checks failed and looks like the following examples:

### Failed Android Device

```json
{
  "allow": false,
  "device": {
    "allow": false,
    "violations": [
      {
        "error": "device_android_api_level_violation",
        "error_description": "Device is required to have API level 33 or higher. Current API level: 11."
      },
      {
        "error": "device_android_encryption_disabled",
        "error_description": "Device is required to have encryption enabled."
      },
      {
        "error": "device_android_patch_level_violation",
        "error_description": "Device is required to have patchlevel 2022-12-01 or higher. Current patch level: 2019-01-01."
      },
      {
        "error": "device_unknwown_app",
        "error_description": "App is not approved by gematik."
      }
    ]
  },
  "security": {
    "allow": true,
    "violations": []
  }
}
```

### Failed iOS Device

```json
{
  "allow": false,
  "device": {
    "allow": false,
    "violations": [
      {
        "error": "device_ios_invalid_version",
        "error_description": "Device is required to have iOS 14.0.0 or higher. Current version: 13.0.0."
      },
      {
        "error": "device_unknown_app",
        "error_description": "App is not approved by gematik."
      }
    ]
  },
  "security": {
    "allow": true,
    "violations": []
  }
}
```