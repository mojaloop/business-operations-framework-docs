# Introduction

The bizOps framework is a framework through which hub operators can build and deploy their business process portals. The framework:
1. Implements a best practice RBAC and IAM integration/implementation.
2. Includes a deployment plan for including the RBAC and IAM solution into Mojaloop
3. Includes a deployment plan for the UI portal so that it can be deployed into a CDN network.
4. Uses micro-frontends that are built from different repositories to decouple community efforts and facilitate easy extension and customisations.
5. Provides an audit trail of all activities performed.

Three levels or degrees of control are required when configuring the best practice security.
1. Daily access to IAM user interfaces where users are created, suspended and their roles assigned.
2. Mappings of roles to permissions that can be edited through a configuration change request.
3. Restrictions on API access on the basis of permissions available to a subject (a user or API client) through their roles.
## Context
### Reference Architecture
The reference architecture workstream has through a collaborative process designed the future / next version architecture of Mojaloop. The BizOps Framework is being designed to work on the current existing Mojaloop version. The BizOps Framework must however be compatible with the reference architecture, and wherever possible, facilitate the move towards the reference architecture design.
There are three elements in BizOps framework project that are directly contributing to the reference architecture building:
1. **Security bounded context.**
Part of this bounded context is being built as part of this workstream.
The split of the frontend into micro-frontends that can be built, tested and released independently; empowering teams building solutions within each bounded context to be able to independently build API functionality and corresponding UI. Customisations and extensions to each bounded context are also easily supported with this design.
2. **Reporting and auditing bounded context.**
Part of this bounded context is being built as part of this workstream.

Here is an overall view of how the operational APIs, experience APIs, and micro-front ends can be combined into micro frontends forming the BizOps framework.

![Architecture Overview Diagram compatible with the Reference Architecture ](/BizOps-Framework-BizOps-Framework.png) 

## IaC 4.xx
The next version of the infrastructure as code project is planned to use a different set of tools than what is currently in use in the Mojaloop Community.
I.e. WS02 with it’s IS-KM and the HAproxy implementations are to be replaced with Keycloak, Ambassador - Envoy tools. This design is compatible with the next IaC version.
