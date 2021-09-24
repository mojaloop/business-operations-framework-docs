# Business Operations Framework Documentation
Join the collaboration for building a “get started quickly” set of core business processes that are easy to customise and contribute to open source, and define and follow best practice.

The bizOps framework is a framework through which hub operators can build and deploy their business process portals. The framework:
1. Implements a best practice RBAC and IAM integration/implementation.
2. Includes a deployment plan for including the RBAC and IAM solution into Mojaloop
3. Includes a deployment plan for the UI portal so that it can be deployed into a CDN network.
4. Uses micro-frontends that are built from different repositories to decouple community efforts and facilitate easy extension and customisations.
5. Provide an audit trail of all activities performed.

## Document project layout structure
The UML diagam's Plant UML scripts reside in the PlantUML folder
The vuepress implementation resides in the docs folder.
 
**Deploy docs locally**
In order to deploy the documents locally 

Change folder to docs
```
cd docs
```
Install dependencies
```
npm install
```
Run the vuepress document development script
```
npm run dev
```
or run
```
Yarn dev
```
The markup files are stored in the 'docs\src\guide' folder.
Images that need to be referenced by the markup can be coppied into 'docs\src\.vuepress\public' folder.
