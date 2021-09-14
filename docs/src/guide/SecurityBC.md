# Security Bounded Context
todo: need an introduction here


## Tools / standards chosen
1. **Ory Oathkeeper** <br>
Will be used as the Identity and Access Proxy (IAP) that will check authentication and authorization before providing access to functional end points. I.e. it will be used to enforce the access control.
2. **Ory Keto** <br>
Will check authorization via subject-role and role-permission mappings. It uses a flexible object, relationship and subject structure pioneered at Google that can model many authorization schemes, including RBAC.
3. **Ory Kratos** <br>
Will use Ory Kratos to create and manage the cookie authorization object.
4. **OpenID Connect** <br>
Is the standard that has been chosen to interact with an identity management system. This is a widely supported standard, and is compatible with all the tools currently in use in the Mojaloop community. I.e. WS02 IS, Keycloak and Ory Kratos

## Overview Architecture

![Architecture Overview Diagram of Security Bounded Context Implementation](/BizOps-Framework-IaC-3.xx-&-Mojaloop-13.xx.png)  

## Tools Services and the roles they are playing
1. **WS02 IS KM**<br>
**owns:** the users<br>
**Implements:** user login redirection and UI that creates the cookie token<br>
**Implements:** standard OIDC authorization code flow
2. **Ory Keto**<br>
**owns:** the roles mapped to users<br>
**represents:** the permissions mapped to roles<br>
**owns:** the participant mapped to users<br>
**Implements:** API RBAC Authorisation check through Oathkeeper<br>
**Implements:** API ABAC Authorisation check through Operational API call
3. **Ory Oathkeeper**<br>
**owns:** the permissions related to API access<br>
**Implements:** API Gateway for operational APIs with authentication and authorization checks<br>
4. **Ory Kratos**<br>
**Owns:** nothing<br>
**Implements:** Authentication Cookie<br>
5. **BC Operational API -**<br>
**owns:** the permissions related to the operational API calls<br>
**Implements:** operational API functions
6. **Shim**<br>
**Owns:** nothing<br>
**Implements:** redirect to configure OIDC<br>
7. **Operator Role + Kubernetes role-resource file**<br>
**Operator Role Owns:** nothing<br>
**Role-resource file Owns:** the roles and the role-permission assignments<br>
**Implements:** update Keto to reflect role-permission assignment changes made in the role-resource file<br>
8. **User Role API**<br>
**Owns:** nothing<br>
**Implements:** role-user API controls (list of users, list of roles, list of roles assigned to users, add role to user, remove role from user.)<br>
**Implements:** participant-user API controls (list of users, list of participants, list of participants assigned to user, add participant to user, remove participant from user.)<br>

## Alignment to the Reference Architecture
![Overview Diagram illustrating the alignment to the Reference Architecture](/BizOps-Framework-Security-BC.png) 

### Functions Implemented
This implmentation of the security BC takes advantage of standard security tools available. In the case where a reference architecture function is alrealy implemented by the chosen standard tool, then it was decided to use that tools function. Where that was done, a mention to the appropriate tool is made.
1. **Create Users / Apps / Groups** <br>
The create of users, groups and application identity accounts should be done directly in the chosen Identity and Key Management solution being used. In the IaC 3.xx version deployment of Mojaloop, this is the WS02 IS KM module.
2. **Login** <br>
A cookie based authentication token has been implemented, and this funcitonallity is performed by both Ory Kratos (who manages the cookie) and the Identity & Key Management system (who create the authentication token).

3. **Assign Users / Apps / Groups to Roles** <br>
The assignment of user, group and app identity accounts to roles is performed by the backend Role-Api, (who make the relevent changes to Keto).
4. **Create Roles & Assign Privileges to Roles** <br>
The assignment of permissions or privileges to roles is performed by change the roleresource.yml files. A version control system e.g. github, is the control mechanism for changes made to this file. Change to the file are uploaded to keto via the kubernetes role operator.
### Functions not Implemented
1. **Communicate Permissions/Privileges** by each BC on startup.<br>
This is not implemented
2. **BC Login**<br>
This refers to the logging in of each bounded context as they startup. Current no bounded context have implemented this functionallity, so this requirement has not been implemented either.
3. **BC setup callback**<br>
There is no requirement for this if each BC doesn't login. This has not been implemented.

### Additional functionallity added
1. **BC verify claim** <br>
By this we are refering to the ability of a Bounded Context that is implementing an Operational API to be able to query if the provided token has authorisation to a defined permission or privilege.<br>
This was not a requirement specified by the reference architecture, however it was a requirement to fulfill the generic reporting BC API. This functionallity has been made available and is implemented using a standard Keto API.
2. **Assignment of Participant access**
In order to implementent external API's to participant DFSPs, an aditional permission assignment was added where participant access is added to identity accounts. This has been implemented in Keto and maintained through the User-Role API. Checking access is through a standard Keto API call.

## Sequence Diagrams
### Logging into the UI
This sequence diagram illustrates the sequence of events that occur when a brower attemps to access a backend API. If the browser is already logged in the request is forwarded. If the brower isn't logged in then a standard Open Id Connect (OIDC) authorization flow is triggered starting with a redirect.
![Sequence diagram illustrating how a brower logs in](/frontend.png) 

### Micro-frontend querying data from operational API
Sequence diagram showing more details regarding valid or invalid bearer tokens, or whether authorisation passes or fails.
![Sequence diagram illustrating how an API client call has its authorisation performed](/client.png) 

### Micro-frontend querying operational API and with additional authorization check
If the operational API requires and additional authorisation check, this is what that sequence would look like.
![Sequence diagram illustrating how an API client call has its authorisation performed](/clientgraphql.png) 

### Assigning Roles & Participant access to Users
This sequence describes how the User Role and User Participation Access user interface would access the User Role API service.
![Sequence diagram illustrating how roles and participant access is assigned to users](/userroles.png) 

### Assigning Permissions to Roles
This shows how changes to the roleresource yml files that map permissions to roles that are stored in a code repository, are acted on by the role permission operator.
![Sequence diagram illustrating how roles and participant access is assigned to users](/rolepermissions.png) 
Example of resource YML file roleresource.yaml

```yml
apiVersion: "mojaloop.io/v1"
kind: MojaloopRole
metadata:
  name: arbitrary-name-here
spec:
  # must match what it is used in Keto, whatever that is
  role: RoleIdentifier
  permissions:
  - permission_01
  - permission_02
  - permission_03
  - permission_04'
```
## Ory Keto - implemention detail
Ory Keto in this design is the tool that implements the logic of whether a login token has the correct authorisation to access an aspect of the system. I.e. it is use to enforce the RBAC (Role Based Access  Control). There are three part to how this is implemented in Keto:
1. The assignment of roles to users.<br>
This functionallity will be maintained and updated from the Role-User API module; which will call and update Keto accordingly.
2. Tha assignment of participant access to a user. <br>
This refers to the DFSP access reports that must only provide reports for the configured participants.
This functionallity will also be maintained via the Role-User API module; which will call and update Keto accordingly. 
3. The assignment of permissions or privileges to roles. This will be controled through the edits of a github roleresource.yml file. The kubernetes role-permission operator is the service that will monitor these roleresource files, and update Keto to affect the assignments.

### Adding roles and participant access to users in Keto
The user list (which includes both people and service accounts) will be retrieved from WSO2 Identity Server, and the participant list from the existing API for that purpose. In both cases, a durable permanent identifier should be what is then used as part of the Keto calls.

The list of roles will be hardcoded, and every role should be given a unique short identifier that is human readable and writeable as well as a name. The UI should display both the identifier and the name, since the identifier will be needed for use in the role-permission operator.

There will be two Keto namespaces used for calls, role and participant. The Keto tuples used will be 
```
role:ROLEID#member@USERID and participant:PARTICIPANTID#member@USERID 
```
(using the notation used for [Keto/Zanzibar](https://www.ory.sh/keto/docs/concepts/relation-tuples). )

The reuse of the relation "member" is not an issue, each relation is namespace-specific. If there's a preferred terminology for the participant-user relationship other than member, that word can be used instead, and should be documented here.

To retrieve the role or participant list for a particular user, the [Query Relation Tuples API](https://www.ory.sh/keto/docs/reference/rest-api#query-relation-tuples) will be used , and the namespace, relation, and subject provided as parameters. This will provide a list of tuples in the response, with a next page token if there are additional results, and the identifiers for the participants or roles can be read from the resulting tuples.

When a role or participant is added or removed for a user, the [create](https://www.ory.sh/keto/docs/reference/rest-api#create-a-relation-tuple) and [delete](https://www.ory.sh/keto/docs/reference/rest-api#delete-a-relation-tuple) relation tuple calls can be used, since only a single tuple is involved at a time. If the call fails, but the failure isn't 4xx, it should be retried a couple times.

### Adding permissions or privileges to roles in Keto
This will be a Kubernetes Operator for a Custom Resource Definition [(CRD)](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/_print/). The operator could be implemented in most any language; existing Modusbox Operator expertise is mostly based around `kopf`, a python framework, but there are options in Go and Node as well (and others).

The operator should maintain a memory of all resources it manages, grouped by role. [Kopf's indexing](https://kopf.readthedocs.io/en/latest/indexing/) functionality is ideal for this: 

When a role resource is changed, the list of permissions for that role across all role resources should be compiled, and a change sent through the Keto Patch [Multiple Relation Tuples API](https://www.ory.sh/keto/docs/reference/rest-api#patch-multiple-relation-tuples) using the patch actions `insert` and `delete`. It is necessary to take all role resources into account, because there may be multiple resources for one role, and more than one may include the same permission, so deleting a role resource that maps Role X to Permission P does not necessarily mean that the Keto tuple for that role-permission connection needs to be deleted, since there may still be another role resource mapping Role X to Permission P.

The Keto tuples will be of the form: 
```
permission:PERMISSIONID#granted@role:ROLEID#member
```

The specific operations on resource change are as follows:

1. Retrieve current permissions granted for role using the [Query Relation Tuples API](https://www.ory.sh/keto/docs/reference/rest-api/#query-relation-tuples)
2. Based on stored index of roles to permissions, compute a diff from the retrieved list.
3. Execute patch from diff.
4. If there are problems, throw an exception so the problem will log and a re-sync will be attempted later.

### How to call the standard Keto API to check authorization
Todo: we need some examples here.
