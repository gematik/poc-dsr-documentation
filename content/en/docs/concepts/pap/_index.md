---
linkTitle: PIP & PAP
title: Policy Information Point & Policy Administration Point
weight: 1
---


## Policy Information Point (PIP)

Contextual information is required to evaluate individual rules of the Zero Trust architecture. In addition to the `Ìdentity Provider`, the ´Device Management Service´ and the `Trust Client`, this information is provided by `Policy Information Points`. `Policy Information Points` can provide information centrally for all services (cross-service `Policy Information Point`) or provide local information for rule evaluation for a specialist service (specialist service-specific `Policy Information Point`). An example of the first case is information about secure device models, which can be included in the evaluation of device security and thus in the access decision. `Policy Information Points` can also be filled with current information from the comprehensive monitoring. In this way, access decisions can be made dynamically based on current information at the time of access.

## Policy Administration Point (PAP)

The rules of a Zero Trust architecture must be created, configured, tested and managed. This function is performed by the `Policy Administration Point`. The set of rules managed via the  `Policy Administration Point` enables attribute-based access control (ABAC). This allows different access requirements to be implemented for different roles, such as service providers or insured persons with different requirements for their end devices and environment.