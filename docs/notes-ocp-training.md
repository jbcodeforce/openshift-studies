# Notes on training

## D0180 

student, which has the password student, root redhat
Student workstation: workstation.lab.example.com
http://rol.redhat.com
Students also have access to a MySQL and a Nexus server hosted by either the OpenShift cluster or by AWS
github.com jbcodeforce
quay.io 

## Container technology

difference between container applications and traditional deployments

* The major drawback to traditionally deployed software application is that the application's dependencies are entangled with the runtime environment
* a traditionally deployed application must be stopped before updating the associated dependencies
        * complex systems to provide high availability 
* A container is a set of one or more processes that are isolated from the rest of the system. 

        * security, storage, and network isolation
        * isolate dependent libraries and run time resources
        * less resources than VM, start quickly.
        * helps with the efficiency, elasticity, and reusability of the hosted applications, and portability
        * [Open Container initiave](https://www.opencontainers.org/)
* Container image: bundle of files and metadata
* Container engine: Rocket, Drawbridge, LXC, Docker, and Podman

* Started in 2001 with VServer, then move to isolated process which leverage the linux features:

        * **Namespaces**: The kernel can isolate specific system resources, usually visible to all processes, by placing the resources within a namespace. Namespaces can include resources like network interfaces, the process ID list, mount points, IPC resources, and the system's host name information.
        * **cgroups**: Control groups partition sets of processes and their children into groups to manage and limit the resources they consume.
        * **Seccomp** defines a security profile for processes, whitelisting the system calls, parameters and file descriptors they are allowed to use
        * SELinux (Security-Enhanced Linux) is a mandatory access control system for processes. Protect processes from each other and to protect the host system from its running processes

## Openshift

RHOCP adds the capabilities to provide a production PaaS platform such as remote management, multitenancy, increased security, monitoring and auditing, application life-cycle management, and self-service interfaces for developers.

Username	RHT_OCP4_DEV_USER	boyerje-us
Password	RHT_OCP4_DEV_PASSWORD	77e54efd355e48418233
API Endpoint	RHT_OCP4_MASTER_API	https://api.ocp-na2.prod.nextcle.com:6443
Console Web Application		https://console-openshift-console.apps.ocp-na2.prod.nextcle.com
Cluster Id		57c48a8d-c97b-433f-86f2-c6c80c516bed

## Compendium

* [Open Container initiave](https://www.opencontainers.org/)
* [Podman, the daemonless container engine for developing, managing, and running OCI Containers on your Linux System](https://podman.io/)
* [Quay image registry](https://quay.io)
* [kubernetes](https://kubernetes.io/)
* [openshift](https://www.openshift.com/)
* [Git commands summary](https://rol.redhat.com/rol/app/apd.html)