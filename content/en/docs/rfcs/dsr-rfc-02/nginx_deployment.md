---
linkTitle: nginx Deployment
title: nginx Deployment
---

## Using nginx as frontend proxy with subrequest

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

