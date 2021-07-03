# OpenShift Studies

OpenShift Container Platform is about developing, deploying, and running [containerized](/docker) applications. It is based on docker and Kubernetes and add the following features:

* **Routes**: represents the way external clients are able to access applications running on OpenShift. The default OpenShift router (HAProxy) uses the HTTP header of the incoming request to determine where to proxy the connection.
* **Deployment config**: Represents the set of containers included in a pod, and the deployment strategies to be used.
* CLI, [REST API](https://docs.OpenShift.org/latest/rest_api/index.html) for administration or Web Console
* Multi tenants. You can also grant other users access to any of your projects. 
* [Source-to-image (S2I)](https://docs.OpenShift.org/latest/creating_images/s2i.html) is a tool for building reproducible Docker images. S2I supports incremental builds which re-use previously downloaded dependencies, and previously built artifacts. OpenShift is S2I-enabled and can use S2I as one of its build mechanisms.
* **Build config**: Used by the OpenShift Source-to-Image (S2I) feature to build a container image from application source code stored in a Git repository

* OpenShift for production comes in several variants:

    * OpenShift Origin: from [http://OpenShift.org](http://OpenShift.org)
    * OpenShift Container Platform: integrated with RHEL and supported by RedHat. It allows for building a private or public PaaS cloud.
    * OpenShift Online: multi-tenant public cloud managed by Red Hat
    * OpenShift Dedicated: single-tenant container application platform hosted on Amazon Web Services (AWS) or Google Cloud Platform and managed by Red Hat.

See also [my summary on k8s](k8s/k8s-0.md).

## Concepts

OpenShift is based on Kubernetes. It adds the concept of **project**, mapped to a k8s namespace, to govern the application access
 control, resource quota and application's life cycle. Project is the top-level element for one to many applications.

We can deploy any docker image, as soon as they are well built: such as defining the port any service is exposed on, not needing to run specifically as the root user or other dedicated user, and which embeds a default command for running the application.

Routes are used to expose app over HTTP. OpenShift can handle termination for secure HTTP connections, or a secure connection can be tunneled through the application, 
with the application handling termination of the secure connection. Non HTTP applications can be exposed via a tunneled secure connection if the client supports
 the SNI extension for a secure connection using TLS.
A router (ingress controller) forwards HTTP and TLS requests to the service addresses inside the Kubernetes SDN.

OpenShift routes are implemented by a cluster-wide router service, which runs as a containerized application in the OpenShift cluster. 
The router service uses HAProxy as the default implementation.

[Red Hat OpenShift on IBM Cloud](https://developer.ibm.com/openlabs/OpenShift) is a preconfigured OpenShift environment available
 for four hours at no charge. 

## Getting started

Use IBM Cloud cluster feature to get an OpenShift cluster, or use [OpenShift online](https://docs.OpenShift.com/online/getting_started/basic_walkthrough.html).

Be familiar with [OC cli commands](oc-cli.md).

## Collaborate

User can be added to an existing project, via the View membership menu on a project. Each user can have different roles. 
`Edit Role` can perform most tasks within the project, except tasks related to administration of the project.

!!! Remark
    State about the current login session is stored in the home directory of the local user running the `oc` command, 
    so user needs to logout and login to access a second cluster. 

