---
linkTitle: DSR-RFC-06 Security Attributes
title: DSR-RFC-06 Device Security Attributes
weight: 6
---

## Introduction

![dev_sec_level](device_security_figure.png)

## Minimal Trust Base for Registration
Minimal/basic trust that is needed for a successful registration at GMS and thus for participating in the DSR. Is verified by GMS during registration process.

### Android

#### Google Play Integrity API
Descriptions are partially taken from the [Android Developers Play Integrity doucmentation](https://developer.android.com/google/play/integrity/verdicts).

<table>
<thead>
  <tr>
    <th>Attribute</th>
    <th>Expected Value</th>
    <th>Description</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td colspan="3"><b>requestDetails:</b></td>
  </tr>
  <tr>
    <td class="identifier">requestPackageName</td>
    <td>is equal to packageName from token payload and from AppIntegrity</td>
    <td>
      <p>application package name the attestation was requested for</p>
      <p>check list with enrolled apps at DMS</p>
    </td>
  </tr>
  <tr>
    <td class="identifier">nonce</td>
    <td>nonce_Integrity</td>
    <td>base64-encoded URL-safe no-wrap nonce provided by the developer</td>
  </tr>
  <tr>
    <td class="identifier">timestampMillis</td>
    <td>t + 10 min < time of creation on device</td>
    <td>timestamp in milliseconds when the request was made</td>
  </tr>
  <tr>
    <td colspan="3"><b>appIntegrity:</b></td>
  </tr>
  <tr>
    <td class="identifier">appRecognitionVerdict</td>
    <td>PLAY_RECOGNIZED</td>
    <td>app and certificate match the versions distributed by Google Play</td>
  </tr>
  <tr>
    <td class="identifier">packageName</td>
    <td>must be in the list of available packages</td>
    <td>
      <p>package name of the app</p>
      <p>check list with enrolled apps at DMS</p>
    </td>
  </tr>
  <tr>
    <td class="identifier">certificateSha256Digest</td>
    <td>must be equal to the sha256 digest, defined in the available packages list</td>
    <td>
      <p>sha256 digest of app certificates</p>
      <p>check list with enrolled apps at GMS</p>
    </td>
  </tr>
  <tr>
    <td class="identifier">versionCode</td>
    <td>must be in the list of available packages</td>
    <td>
      <p>version of the app</p>
      <p>check list with enrolled apps at DMS</p>
    </td>
  </tr>
  <tr>
    <td colspan="4"><b>deviceIntegrity:</b></td>
  </tr>
  <tr>
    <td class="identifier">deviceIntegrity</td>
    <td>MEETS_DEVICE_INTEGRITY</td>
    <td>app is running on an Android device powered by Google Play services, device passes system integrity checks and meets Android compatibility requirements</td>
    <td></td>
  </tr>
  <tr>
    <td colspan="4"><b>accountDetails:</b></td>
  </tr>
  <tr>
    <td class="identifier">appLicensingVerdict</td>
    <td>LICENSED</td>
    <td> user has an app entitlement (user installed or bought your app on Google Play)</td>
    <td></td>
  </tr>
</tbody>
</table>

#### Android Key & ID Attestation
Descriptions are partially taken from the [Android Developers Key & ID Attestation article](https://developer.android.com/privacy-and-security/security-key-attestation).
<table>
<thead>
  <tr>
    <th>Attribute</th>
    <th>Expected Value</th>
    <th>Description</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td colspan="3"><b>KeyDescription:</b></td>
  </tr>
  <tr>
    <td class="identifier">attestationVersion</td>
    <td>tbd</td>
    <td>version of the key attestation feature. </td>
  </tr>
  <tr>
    <td class="identifier">attestationSecurityLevel</td>
    <td>TrustedEnvironment (1), StrongBox (2)</td>
    <td>security level of the attestation</td>
  </tr>
  <tr>
    <td class="identifier">keyMintVersion / keymasterVersion</td>
    <td>tbd</td>
    <td>security level of the attestation</td>
  </tr>
  <tr>
    <td class="identifier">keyMintSecurityLevel  / keymasterSecurityLevel</td>
    <td>tbd</td>
    <td>security level of the Keymaster/KeyMint implementation</td>
  </tr>
  <tr>
    <td class="identifier">attestationChallenge</td>
    <td>nonce_keypair_attest</td>
    <td>challenge from creation</td>
  </tr>
  <tr>
    <td class="identifier">softwareEnforced</td>
    <td>out of scope for PoC</td>
    <td></td>
  </tr>
  <tr>
    <td class="identifier">teeEnforced</td>
    <td>out of scope for PoC</td>
    <td></td>
  </tr>
</tbody>
</table>

### iOS
#### App Attest Service
Descriptions are partially taken from the [Apple Developer DeviceCheck documentation](https://developer.apple.com/documentation/devicecheck/preparing_to_use_the_app_attest_service).
<table>
<thead>
  <tr>
    <th>Attribute</th>
    <th>Expected Value</th>
    <th>Description</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td colspan="4"><b>Attestation:</b></td>
  </tr>
  <tr>
    <td>RP ID (32 bytes)</td>
    <td>must be equal to the RP ID, defined in the available packages list at DMS</td>
    <td>A hash of your app’s App ID, which is the concatenation of your 10-digit team identifier, a period, and your app’s CFBundleIdentifier value</td>
  </tr>
  <tr>
    <td>counter (4 bytes)</td>
    <td>ignored for PoC</td>
    <td>value that reports the number of times your app has used the attested key to sign an assertion</td>
  </tr>
  <tr>
    <td>aaguid (16 bytes)</td>
    <td>production</td>
    <td>App Attest–specific constant that indicates whether the attested key belongs to the development or production environment</td>
  </tr>
  <tr>
    <td>credentialId (32 bytes)</td>
    <td>must be equal to the key used to sign the mTLS public key</td>
    <td>hash of the public key part of the attested cryptographic key pair</td>
  </tr>
</tbody>
</table>

## Device Rating Attributes
Device security attributes that need to be provided by a device when trying to access a resource. GMS verifies token authenticity / integrity as well as app/Trust SDK info and forwards all information in device_token to PEP.
### Android
#### Google Play Integrity API
see [Minimal Trust Base for Registration]({{< relref "_index.md#google-play-integrity-api" >}}).
#### Android Key & ID Attestation
see [Minimal Trust Base for Registration]({{< relref "_index.md#android-key--id-attestation" >}}).

#### Additional Security Attributes
Descriptions are partially taken from the [Android Enterprise Developers Zero Trust signals documentation](https://developers.google.com/android/work/zero-trust-signals).
| Attribute | Description | API | Root of Trust | Availability |
|---|---|---|---|---|
| **Android version**  |  Android version or API level / SDK version currently running on the device |  ```Build.VERSION.SDK_INT``` | Software |  >= Android 1.6 |
| **Android version (release)**  |  Android version (API level) with which the device was released / CTS was passed |  ```getprop('ro.product.first_api_level')``` | Software | TODO |
| **Patchlevel** | OS patch level |  ```Build.VERSION.SECURITY_PATCH```  | Software | >= Android 6.0 |
| **FDE / FBE** |  Indicates whether device encryption is supported and whether it is activated. |  ```getprop('ro.crypto.state')```  | Software | TODO |
| **System PIN / password / pattern set** | Indicates whether a PIN/pattern/password is set for the lock screen. |  ```KeyguardManager.isDeviceSecure()```, ```BiometricManager.canAuthenticate(BiometricManager.Authenticators.DEVICE_CREDENTIAL)```, ```BiometricManager.canAuthenticate(BiometricManager.Authenticators.BIOMETRIC_STRONG)``` | Software | >= Android 6.0,  >= Android 11, >= Android 12 |
| **System PIN / password / pattern quality** | The Device Policy Manager can be used to query whether certain password complexity levels are currently being met. |  ```DevicePolicyManager.getPasswordComplexity()```, requires ```android.permission.REQUEST_PASSWORD_COMPLEXITY``` | Software | >= Android 10 |
| **Verified boot supported** | Indicates whether VerifiedBoot is available on the device. |  ```PackageManager.FEATURE_VERIFIED_BOOT```  | Software | >= Android 5.0  |
| **Mainline patch level** |  Indicates when the last mainline patch was installed. |  ```PackageManager.getPackageInfo("com.google.android.modulemetadata", 0).versionName``` | Software | API level > 1 |
| **OEM / model** | Returns information about manufacturer, model, etc.  |  ```BUILD.MODEL, BUILD.PRODUCT, BUILD.MANUFACTURER, BUILD.BOARD```  | Software |  |
| **Biometric class** |Returns information if class 3 biometrics is available.  |   ```BiometricManager.canAuthenticate(Authenticators#BIOMETRIC_STRONG)``` | Software | >= Android 12 |

### iOS
#### App Attest Service

Descriptions are partially taken from the [Apple Developer DeviceCheck documentation](https://developer.apple.com/documentation/devicecheck/preparing_to_use_the_app_attest_service).
<table>
<thead>
  <tr>
    <th>Attribute</th>
    <th>Expected Value</th>
    <th>Description</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td colspan="4"><b>Assertion:</b></td>
  </tr>
  <tr>
    <td>RP ID (32 bytes)</td>
    <td>must be equal to the RP ID, defined in the available packages list at DMS</td>
    <td>A hash of your app’s App ID, which is the concatenation of your 10-digit team identifier, a period, and your app’s CFBundleIdentifier value</td>
  </tr>
  <tr>
    <td>counter (4 bytes)</td>
    <td>ignored for PoC</td>
    <td>value that reports the number of times your app has used the attested key to sign an assertion</td>
  </tr>
    <td colspan="4"><b>Fraud Risk (optional):</b></td>
  </tr>
  </tr>
    <td colspan="4"><b>tbd</b></td>
  </tr>
</tbody>
</table>


#### Additional Security Attributes
| Attribute  | Description | API  | Root of Trust | Availability |
|---|---|---|---|---|---|
|  System Name	 |  The name of the operating system running on the device. |  `UIDevice: var systemName: String { get }` |  Software | >= iOS 2.0   | 
| System version  |  The current version of the operating system. |   `UIDevice: var systemVersion: String { get }` | Software |	 >= iOS 2.0  |
| Model  | Possible examples of model strings are ”iPhone” and ”iPod touch”. | `UIDevice: var model: String { get }` |Software  | 	 >= iOS 2.0  |
| identifierForVendor  | An alphanumeric string that uniquely identifies a device to the app’s vendor. | `UIDevice: var identifierForVendor: UUID? { get }` | Software  |  >=iOS 6.0 |
| App Version  | The current version of the App system.  | tbd | Software  | tbd  |