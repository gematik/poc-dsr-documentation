---
linkTitle: "DSR-RFC-11 MVM"
title: "DSR-RFC-11 Mobile Vulnerability Management"
date: 2023-11-09T17:19:29+01:00
draft: true
---

{{% pageinfo %}}
Content is under development
{{% /pageinfo %}}

## Introduction
tbd

![mvm_pipeline](mvm_pipeline.png)

## Example Vulnerabilities

| Published | Title  | CVEs | Severity | Description | Affected Devices | Rule(s) for DSR | Needed Information to detect | CPE (via NVD)  | References |
|---|---|---|---|---|---|---|---|---|---|
| <ul><li>Spring 2022</li><li>Summer 2022</li></ul> | Mali GPU Kernel Driver may elevate CPU RO pages to writable | <ul><li>CVE-2021-39793</li><li>CVE-2022-22706</li><li>CVE-2022-33917</li><li>CVE-2022-36449</li><li>(2325, 2327, 2331, 2333, 2334) </li></ul> | <ul><li>MEDIUM</li><li>HIGH</li></ul> | A non-privileged user can get a write access to read-only memory pages; by forcing the kernel to reuse these pages as page tables, an attacker with native code execution in an app context could gain full access to the system, bypassing Android's permissions model and allowing broad access to user data. | Devices with Mali GPU: <ul><li>Pixel</li><li>Xiaomi</li><li>Oppo</li><li>...</li></ul> | WARN all devices with MALI GPU && tbd Patchlevel | <ul><li>GPU manufacturer and model</li><li>Android Security Bulletin </li></ul> | nn | https://googleprojectzero.blogspot.com/2022/11/mind-the-gap.html |
| Spring 2022 |  Samsung TrustZone Keymaster | <ol><li>CVE-2021-25444</li><li>CVE-2021-25490</li></ol> | HIGH | KeyMaster in TEE: <ol><li>IV reuse attack on AES-GCM that allows an attacker to extract hardware-protected key material</li><li>downgrade attack that makes even the latest Samsung devices vulnerable to the IV reuse attack</li></ol> | <ol><li> Galaxy S8, S9, S10, S20, and S21</li><li>all Samsung devices released with < Android P</li></ol>  |  <ol><li>DENY all SAMSUNG devices with SAMSUNG_PATCHLEVEL < SMR AUG-2021 Release 1</li><li>DENY all SAMSUNG devices with SAMSUNG_PATCHLEVEL < SMR Oct-2021 Release 1</li></ol> |  <ul><li>Samsung patch level information (ro.build.XXXX)</li><li>Samsung SMR</li></ul> | <ol><li>cpe:2.3:o:google:android:8.1:*:*:*:*:*:*:*; cpe:2.3:o:google:android:9.0:*:*:*:*:*:*:*; cpe:2.3:o:google:android:10.0:*:*:*:*:*:*:*</li><li>cpe:2.3:o:google:android:9.0:*:*:*:*:*:*:*; cpe:2.3:o:google:android:10.0:*:*:*:*:*:*:*; cpe:2.3:o:google:android:11.0:*:*:*:*:*:*:*</li></ol> |  <ul><li>https://eprint.iacr.org/2022/208.pdf</li><li>https://www.schneier.com/blog/archives/2022/03/samsung-encryption-flaw.html</li><li>https://security.samsungmobile.com/securityUpdate.smsb?year=2021&month=8</li></ul> |
|  Spring 2023 | Internet to Baseband Remote Code Execution Vulnerabilities in Exynos Modems | <ol><li>CVE-2023-24033</li><li>CVE-2023-26496</li><li>CVE-2023-26497</li><li>CVE-2023-26498</li><li>CVE-2023-26072</li><li>CVE-2023-26073</li><li>CVE-2023-26074</li><li>CVE-2023-26075</li><li>CVE-2023-26076</li><li>...</li></ol> | HIGH  | 1.-4.: allow an attacker to remotely compromise a phone at the baseband level with no user interaction, and require only that the attacker know the victim's phone number <br/> 5.-??.: require either a malicious mobile network operator or an attacker with local access to the device| Devices with Exynos 1280, 2200, 5300 modem (CVE-2023-28613):<ul><li>Google Pixel 6, 7</li><li>Samsung S22, M33, M13, M12, A71, A53, A33, A21s, A13, A12 and A04 series</li><li>Vivo S16, S15, S6, X70, X60 and X30</li><li>(Exynos Auto T5123)</li></ul> | 1.-4: <ul><li>WARN* GOOGLE_PIXEL models 6,7 with PATCHLEVEL < March 2023 </li><li>WARN* Samsung models S22, M33, ...</li><li>WARN* Vivo models S16, S15, ...</li> * turning off Wi-Fi calling and Voice-over-LTE (VoLTE) | <ul><li>device manufacturer & model </li><li>Android patch level</li><li>modem manufacturer & model </li><li>Android Security Bulletin</li><li>Samsung SMR</li> </ul> | <ul><li>cpe:2.3:o:samsung:exynos_1280_firmware:-:*:*:*:*:*:*:* on cpe:2.3:o:samsung:exynos_2200_firmware:-:*:*:*:*:*:*:*</li><li>cpe:2.3:o:samsung:exynos_2200_firmware:-:*:*:*:*:*:*:* on cpe:2.3:h:samsung:exynos_2200:-:*:*:*:*:*:*:*</li><li>cpe:2.3:o:samsung:exynos_modem_5300_firmware:-:*:*:*:*:*:*:* on cpe:2.3:h:samsung:exynos_modem_5300:-:*:*:*:*:*:*:*</li></ul>  | <ul><li>https://googleprojectzero.blogspot.com/2023/03/multiple-internet-to-baseband-remote-rce.html</li><li>https://semiconductor.samsung.com/support/quality-support/product-security-updates/</li><li>https://source.android.com/docs/security/bulletin/pixel/2023-03-01</li></ul>  |
|  Spring 2023 | Smartphone Fingerprint Authentication Brute-force Attack  | nn  | HIGH | Smartphone Fingerprint Authentication to Brute-force Attack with physical access | <ul><li>Xiaomi Mi 11 Ultra (Android 11)</li><li>Vivo X60 Pro (Android 11)</li><li>OnePlus 7 Pro (Android 11)</li><li>Oppo Reno Ace (Android 10)</li><li>Samsung Galaxy  S10+ (Android 9)</li><li>OnePlus 5T (Android 8)</li><li>Huawei Mate30 Pro 5G (HarmonyOS 2)</li><li>Huawei P40 (HarmonyOS 2)</li></ul>  |  tbd | <ul><li>Android version</li><li>Android version</li></ul>  | nn | https://arxiv.org/pdf/2305.10791.pdf  |


## Implementation

The implementation of the pipeline was carried out using the following resources:
* [cpe-guesser](https://github.com/cve-search/cpe-guesser)
* [CVE search library](https://nvdlib.com/en/latest/)

The main.py file receives a list of "keywords" such as "python3 main.py samsung galaxy s6". These keywords are searched for in a local redis-server instance, which contains all CPEs from NIST.

The found CPEs are formatted properly and queried with the nvdlib at the NIST CVE API. The found CVEs are processed and outputted.