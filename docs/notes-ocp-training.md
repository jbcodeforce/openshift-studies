# Notes on training

## DO180 

student, which has the password student, root redhat
Student workstation: workstation.lab.example.com
http://rol.redhat.com
Students also have access to a MySQL and a Nexus server hosted by either the OpenShift cluster or by AWS
github.com jbcodeforce
quay.io jbcodeforce

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
Password	RHT_OCP4_DEV_PASSWORD	<>
API Endpoint	RHT_OCP4_MASTER_API	https://api.ocp-na2.prod.nextcle.com:6443
Console Web Application		https://console-openshift-console.apps.ocp-na2.prod.nextcle.com
Cluster ID ...

Container images are named based on the following syntax: `registry_name/user_name/image_name:tag`

Example of images used:

```shell
sudo podman run ubi7/ubi:7.7 echo "Hello!"
# Apache http server
sudo podman run -d rhscl/httpd-24-rhel7:2.4-36.8
# Get IP address of a last container started
podman inspect -l -f "{{.NetworkSettings.IPAddress}}" 
# mysql
sudo podman run --name mysql-basic -v /var/local/mysql:/var/lib/mysql/data \
> -e MYSQL_USER=user1 -e MYSQL_PASSWORD=mypa55 \
> -e MYSQL_DATABASE=items -e MYSQL_ROOT_PASSWORD=r00tpa55 \
> -d rhscl/mysql-57-rhel7:5.7-3.14
# Stop and start a container
sudo podman stop my-httpd-container
sudo podman restart my-httpd-container
# Send a SIGKILL
sudo podman kill my-httpd-container
```

Quay.io introduces several exciting features, such as server-side image building, fine-grained access controls, and automatic scanning of images for known vulnerabilities.

To configure registries for the podman command, you need to update the `/etc/containers/registries.conf`. The podman search command finds images from all the registries listed in this file.

```
[registries.search]
registries = ["registry.access.redhat.com", "quay.io"]
```

Existing images from the Podman local storage can be saved to a .tar file using the podman save command.

## Compendium

* [Open Container initiave](https://www.opencontainers.org/)
* [Podman, the daemonless container engine for developing, managing, and running OCI Containers on your Linux System](https://podman.io/)
* [Quay image registry](https://quay.io)
* [kubernetes](https://kubernetes.io/)
* [openshift](https://www.openshift.com/)
* [Git commands summary](https://rol.redhat.com/rol/app/apd.html)