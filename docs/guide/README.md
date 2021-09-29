# Introduction

Join the collaboration for building a **“get started quickly”** set of core business processes that are easy to customise and contribute to open source and follow best practice. 

The bizOps framework is a framework through which hub operators can build and deploy their business process portals to support their business processes as defined the in the [Mojaloop business documentation](https://docs.mojaloop.io/mojaloop-business-docs/). The Business Operations Framework supports community collaboration in building a User Experience for a Mojaloop hub operator that includes robust APIs, follows best practices and is secure by design that further supports adoption and enhancing off-the-shelf value of the Mojaloop solution.

The resulting UI is not intended to be comprehensive, but to demonstrate an exemplar web experience that is easy to extend and customize. It is therefore important that Role Based Access Control, interfacing with standard Identity Access Management systems, API level security control, micro-frontends and maker-checker workflows are supported. The UX architecture follows a pre-compiled bundle with a backing API pattern that can be deployed on a Content Delivery Network (CDN). 

This design documentation provides a more detailed designs that will include security aspects, technologies used and architecture patterns.

The framework:
1. Implements a best practice RBAC (Role based access control) and IAM (Identity access management) integration/implementation.
2. Includes a deployment plan for including the RBAC and IAM solution into Mojaloop
3. Includes a deployment plan for the UI (User interface) portal so that it can be deployed into a CDN (Content Delivery Network) network.
4. Uses micro-frontends that are built from different repositories to decouple community efforts and facilitate easy extension and customisations.
5. Provides an audit trail of all activities performed.

Three levels or degrees of control are required when configuring the best practice security.
1. Daily access to IAM user interfaces where users are created, suspended and their roles assigned.
2. Mappings of roles to permissions that can be edited through a configuration change request.
3. Restrictions on API access on the basis of permissions available to a subject (a user or API client) through their roles.

## Reference Architecture - Context
The reference architecture workstream has through a collaborative process designed the future / next version architecture of Mojaloop. The BizOps Framework is being designed to work on the current existing Mojaloop version. The BizOps Framework must however be compatible with the reference architecture, and wherever possible, facilitate the move towards the reference architecture design.
There are three elements in BizOps framework project that are directly contributing to the reference architecture building:
1. **Security bounded context.**
Part of this bounded context is being built as part of this workstream.
The split of the frontend into micro-frontends that can be built, tested and released independently; empowering teams building solutions within each bounded context to be able to independently build API functionality and corresponding UI. Customisations and extensions to each bounded context are also easily supported with this design.
2. **Reporting and auditing bounded context.**
Part of this bounded context is being built as part of this workstream.

Here is an overall view of how the operational APIs, experience APIs, and micro-front ends can be combined into micro frontends forming the BizOps framework.

![Architecture Overview Diagram compatible with the Reference Architecture ](/BizOps-Framework-BizOps-Framework.png) 

The initial delivery of this framework includes a thin vertical slice to demonstrate the end to end functional implementation of the framework. Although this function serves an important puropse, this is not the end objective of this project. This objective is to provide a framework that other community efforts can contribute to. Here is the current todo list of micro-frontends are are intended to be added to this framework by Mojaloop community implementation efforts:
1. **Platform Configuration**
Process to configure the platform so that it enforces the scheme and the scheme rules.
1. **Platform Management**
Technical operational management controls for the platform.
1. **Liquidity Management**
Process support for managing Liquidity.
1. **Participant Lifecycle Management (Hub view)** 
Manage the onboarding and status transitions of participants.
1. **Participant Lifecycle Management (DFSP view)** 
Allow DFSP to manage their status and interaction with the Hub.
1. **Settlement Management**
Management interface for settlement.
1. **Transfer and Transaction Management**
1. **Agreement (Quoting) Management**
1. **Account Lookup & Discovery Management**
1. **3rd Party Initiated Payments Management**
1. **Fees (interchange and billing) Management**
1. **Reporting and analytics**

## IaC 4.xx - Context
The next version of the infrastructure as code project is planned to use a different set of tools than what is currently in use in the Mojaloop Community.
I.e. WS02 with it’s IS-KM and the HAproxy implementations are to be replaced with Keycloak, Ambassador - Envoy tools. This design is compatible with the next IaC version.
