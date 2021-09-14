# Micro-frontend - JAMStack design
## Overview
Overview Diagram showing the deployment of the micro frontends to a CDN.
::: warning Note:
The deployment of the bounded context API is not covered in this diagram.
:::
![Overview diagram showing deployment](/BizOps-Framework-Micro-frontend-deploy.png)

#### High level sequence diagram illustrating how the microservices are loaded.
![High level sequence diagram illustrating how the microservices are loaded.](/microfrontendloading.png)

## Technology Stack


1. **React** 
The framework is based on the React library.
This is the most popular single page application library in use, and additionally this choice allows us to capitalise on other community efforts facilitating an easy conversion into this library.
It can be enhanced by using state container libraries (Redux, Flux, MobX), however there is no restriction on a specific one to use.
The microfrontends come with preconfigured, isolated Redux stores.

2. **Webpack 5** 
Webpack 5 is currently the only javascript bundler that supports remote build separation. It is done by using the Module Federation Plugin.
It enables runtime composition to provide a smooth and fully transparent user experience to the users, resulting in a traditional Single Page Application.
There are additional benefits over other technologies, all that resulting in a small footprint and overall better experience for the users.
Will Implement the host / child micro frontends integration at runtime.

3. **CI/CD and Atomic Deployments** (e.g. Github Actions) 
Each implementation of the BizOps Framework will need to implement their own atomic deployment solution.
The standard BizOps project will use Github Actions to perform the task of executing continuous integration pipeline, running the relevant tests, building the individual micro frontend and deploying the resulting static files over a CDN and/or creating a Docker Image.
Each micro frontend is released in complete autonomy: the composed application can use the updated versions of each individual micro frontend automatically, without involving any further coordination.

4. **Running on a CDN** - Content Delivery Network 
The micro frontends can run on a CDN. Individual builds are composed of only static files (HTML, CSS, Javascript) and can be deployed in different locations / different URLs.
As long as they are available over a secure connection (HTTPS) they can be served from any location and also from different CDNs.

5. **Running in Kubernetes** 
The micro frontends can run in a Kubernetes environment. There are two approaches that can be taken here:
   - The individual micro frontends, and the shell application are containerized (using e.g. Docker) and then hosted in Kubernetes.
The host and the children apps can be deployed on the same cluster or on different clusters as long as they are publicly accessible.
   - Deploy a private CDN into the Kubernetes cluster, and host the static markup files on the CDN.
There are various CDNs available that are compatible with Kubernetes.

## Webpack building

The host and the children apps include scripts to build the distribution artifacts.
The build can be done in the developer host machine, in the CI and in Docker.
## Micro frontend loading 

The host is responsible for loading the children apps at runtime.
It gathers information about the available children at runtime, from either an api or a registry.

It includes an internal engine responsible for loading only the necessary children when they need to be displayed.

The individual micro frontends won’t be loaded when not necessary (e.g. when a specific page is not accessed by the user).


## Repository of micro frontends
In order to provide a centralized authority responsible for controlling the individual micro frontends meeting the necessary requirements, it is suggested to build a solution that works as a registry.

The registry would serve the following purposes:

1. allow the community to register the micro frontends and specify some details
2. expose an api used by the host to retrieve informations about the available micro frontends
3. provide informations around the versions of the available micro frontends

## Why is this design JAMStack and why is that important?
This framework is JAMStack compatible. The deployment will:
1. use Static Markup that is rendered during or before a deployment
2. use API’s to provide functionality and content to the UI
3. use client side Javascript to provide local view and controller functionality and state
use a CDN deploy 
4. implement atomic deploys that are triggered using git actions
5. use automatic cache invalidation by incrementing versions.

## Deployments

The micro frontends use atomic deployments and no full-build is ever required.
Each individual micro frontend deploys independently from the other ones.
## CI/CD Continuous Integration / Continuous Delivery 

Each micro frontend has its own CI/CD setup; there is no requirement to share the same setup or use the same CI tool.

The CI/CD can be configured in order to support multiple environments e.g. DEV, QA, PROD.
Here is an example file showing a git action workflow. ...
## CDNs

The resulting SPA is served by a CDN or multiple CDNs. Individual micro frontends can live in different CDNs.
## Kubernetes

The resulting SPA can run and be served in one or more Kubernetes environments.
### Host application
The host application comes with a pre-configured setup out of the box. It doesn’t need any particular configuration different from a traditional SPA more than the Webpack 5 Module Federation configuration.

It will be acting as the orchestrator, loading the remote micro frontends and providing them with app-wide functionality e.g. auth, RBAC, client side routing.

There is virtually no limit on how the host can grow and how much can be extended.
It is suggested however to centralize all the host-child communication and shared components in an external library so that both host and children have the same knowledge and integration won’t break.

