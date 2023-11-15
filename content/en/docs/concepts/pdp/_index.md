---
linkTitle: PDP
title: Policy Decision Point
weight: 1
---

{{% pageinfo %}}
Content is under development
{{% /pageinfo %}}

## Introduction
tbd

## Realisation of PDP with Open Policy Agent

The PDP in DSR has been implemented using Open Policy Agent (OPA). On the one hand, the architecture of OPA offers numerous flexible integration scenarios (SDK, microservice, proxy); on the other hand, the policy bundles and external data can be well separated. The latter gives us great flexibility in the design and distribution of policies:

Logical policy expressions are distributed as versioned bundles via the "policy as code" approach.
Additional data necessary for the access decision can be updated separately and thus decoupled in time.
This separation allows additional assurance that only tested bundles are installed in production, while dynamic external data can be updated at runtime. OPA supports different strategies for updating external data, see https://www.openpolicyagent.org/docs/latest/external-data/.

## DSR Policy

The DSR policy consists of a list of policy rules that are dynamically evaluated for each request by the PDP. The policy is structured into different modules. This enables policy authors and domain experts to focus on specific parts of the policy. The modules are then compiled together for deployment and are provided as one policy file in form of an OPA bundle to the PDP.

The following policy modules are available:
* Device policy
    * Android
    * iOS
* Security policy

**Note**: Since the DSR Project ist not integrated into a real IDP, there is no user policy.

The device policy contains minimum requirements for users' devices before server access if allowed. Apple App Attest, Google Key & ID Attestation as well as Google Play Integrity attestation services provide different information in their attestation results – this means that different information can be verified in the policy:
* Name and version of the app (Android and iOS)
* Device model (Android and iOS)
* Minimum OS Version (Android and iOS) and security patch level (Android only)
* Attestation security level (Android only)
* Password complexity (Android only)
The security policy is used to identify access from banned IP address, countries or from banned users.

## Policy Structure
Policies habe been broken down into two seperate files. The policy file itself (available in the PAP) and the corresponding data file (PIP Data). This decision was taken with the idea that the policy file is relative static but the data file contains much more dynamic information that has to be updated reguraly.

## Policy Distribution

1. Policy bundles are created and signed via a CI/CD pipeline.
2. Bundles are published via the Bundles Registry. OPA supports OCI or plain HTTPS for this purpose.
3. The service provider replicates the policies from the bundles HTTP server to their local environment. 
4. The service administrator needs to ensure that the PDP/PEP receives the necessary policy bundles.

![dev_sec_level](concept_pdp_overview.png)
