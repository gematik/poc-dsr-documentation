---
linkTitle: DSR-RFC-02
title: DSR-RFC-02 Resource Access
weight: 2
---

{{% pageinfo %}}
Content is under development
{{% /pageinfo %}}

## Main flow

```plantuml
autonumber "<b>[0]"
skinparam defaultFontSize 10
skinparam defaultFontName Helvetica
skinparam DefaultMonospacedFontName Courier
skinparam lengthAdjust none

activate Frontend

participant Service
participant GMS
participant IDP

Frontend -> Service ++: ""GET /resource""
Service -> Service: missing token\nor session expired
Service -> Frontend --: ""401 Unauthorized""\n""WWW-Authenticate: ...""

== Authentication and authorization ==

Frontend -> Service ++: Start authorization
Service -> Frontend --: ""code_challenge"" device\n""code_challenge"" subject

Frontend -> GMS ++: authenticate device
GMS -> Frontend --: ""auth_code"" for device

Frontend -> IDP ++: authenticate subject
IDP -> Frontend --: ""auth_code"" for subject

Frontend -> Service ++: authorize access\n""auth_code device""\n""auth_code"" subject
Service -> GMS ++: get device_token\ncode=..&code_verifier=..
GMS -> Service --: device_token
Service -> IDP ++: get id_token\ncode=..&code_verifier=..
IDP -> Service --: id_token
Service -> Service: create session
Service -> Frontend: ""access_token""

== Session established ==

Frontend -> Service: ""GET /resource""\n""Authorization: Bearer {access_token}""
Service -> Frontend: resource

Frontend -> Service: ""GET /other-resource""\n""Authorization: Bearer {access_token}""
Service -> Frontend: other resource
```

1. `Frontend` requests a resource at a `Service` with missing, expired or invalid `access_token`
1. `Service` verifies the request and determines, that authorization is required
1. `Service` responds with `401` and provides the information for the client how to proceed with authorization using the `WWW-Authenticate` header, see [RFC9110, Section 11.6.1](https://datatracker.ietf.org/doc/html/rfc9110.html#name-www-authenticate)
1. `Frontend` starts the authorization process. At this point it should be known which GMS and which IDP is to be used for authorization, e.g. by configuration or user choice.
1. `Service` generates the code challenges for the device and user subject as described in [RFC7636](https://datatracker.ietf.org/doc/html/rfc7636)
1. `Frontend` initiates and performs the device authorization at `GMS`
1. `GMS` provides the `auth_code` for the device, redeemable only by the `Service`
1. `Frontend` initiates and performs the subject authorization at `IDP`. This step will require a more complex protocol, especially involving some kind of Authenticator and Credentials.
1. `GMS` provides the `auth_code` for the device, redeemable only by the `Service`
1. `Frontend` transmits both `auth_codes` (device and subject) to the token endpoint of the `Service`
1. Service requests `device_token` using the `auth_code` and the `code_verifier` for device
1. GMS issues the `device_token`and transmits it to the Service
1. Service requests `id_token` the `auth_code` and the `code_verifier` for subject
1. IDP issues the `id_token` and transmits it to the `Service`
1. `Service` issues an `access_token` for a limited period of time, establishing the user session with the `Frontend`
1. `Service` transmits `access_token`to the `Frontend` 
1. `Frontend` can now access the resources at the `Service`

## Using nginx with `auth_request` as frontend proxy

In our sample implementation we use nginx as frontend proxy. Each request is authorized using the [`auth_request` subrequest](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html), which is handled by the PEP. If PEP returns any other response as `200`, the proxy access to the business backend will be denied by the nginx. 

```plantuml
autonumber "<b>[0]"
skinparam defaultFontSize 10
skinparam defaultFontName Helvetica
skinparam DefaultMonospacedFontName Courier
skinparam lengthAdjust none

activate Frontend

box "Policy Enforcement"
participant "nginx" as Proxy
participant "dsr-pep" as PEP #Orange
end box

participant "dsr-fahdienst" as Backend

group Unauthorized
Frontend -> Proxy ++: ""GET /resource""
Proxy -> PEP ++: auth subrequest\n""GET /.../auth_request""
PEP -> PEP: missing token\nor session expired
PEP -> Proxy --: ""401 Unauthorized""\n""WWW-Authenticate: ...""
Proxy -> Frontend --: ""401 Unauthorized""\n""WWW-Authenticate: ...""
end

group Authorized
Frontend -> Proxy ++: ""GET /resource""\n""Authorization: Bearer {access_token}""
Proxy -> PEP ++: auth subrequest\n""GET /.../auth_request""
PEP -> PEP: ""access_token"" is valid
PEP -> Proxy --: ""200 OK""
Proxy -> Backend ++: ""proxy_pass GET /resource""
Backend -> Proxy --: Resource
Proxy -> Frontend --: Resource
end


deactivate Frontend 

```

## Policy Enforcement

```plantuml
autonumber "<b>[0]"
skinparam defaultFontSize 10
skinparam defaultFontName Helvetica
skinparam DefaultMonospacedFontName Courier
skinparam lengthAdjust none

activate Frontend

box "Service"
participant PEP #Orange
participant PDP #Orange
participant "Business Backend" as Backend
end box

Frontend --> PEP: request1 

PEP --> PDP ++
PDP --> PEP --

PEP --> Backend ++
Backend --> PEP --

PEP --> Frontend --

```
