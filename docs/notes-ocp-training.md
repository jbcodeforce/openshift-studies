# Notes on training

## DO180 

student, which has the password student, root redhat
Student workstation: workstation.lab.example.com
http://rol.redhat.com
Students also have access to a MySQL and a Nexus server hosted by either the OpenShift cluster or by AWS
github.com jbcodeforce
quay.io jbcodeforce

## Container technology

Difference between container applications and traditional deployments

* The major drawback to traditionally deployed software application is that the application's dependencies are entangled with the runtime environment
* a traditionally deployed application must be stopped before updating the associated dependencies
        * complex systems to provide high availability 
* A container is a set of one or more processes that are isolated from the rest of the system. 

        * security, storage, and network isolation
        * isolate dependent libraries and run time resources
        * less resources than VM, start quickly.
        * helps with the efficiency, elasticity, and reusability of the hosted applications, and portability
        * [Open Container initiative](https://www.opencontainers.org/)
* Container image: bundle of files and metadata
* Container engine: Rocket, Drawbridge, LXC, Docker, and Podman

* Started in 2001 with VServer, then move to isolated process which leverages the linux features:

        * **Namespaces**: The kernel can isolate specific system resources, usually visible to all processes, by placing the resources within a namespace. Namespaces can include resources like network interfaces, the process ID list, mount points, IPC resources, and the system's host name information.
        * **cgroups**: Control groups partition sets of processes and their children into groups to manage and limit the resources they consume.
        * **Seccomp** defines a security profile for processes, whitelisting the system calls, parameters and file descriptors they are allowed to use
        * SELinux (Security-Enhanced Linux) is a mandatory access control system for processes. Protect processes from each other and to protect the host system from its running processes

## OpenShift

RHOCP adds the capabilities to provide a production PaaS platform such as remote management, multi tenancy, increased security, monitoring and auditing, application life-cycle management, and self-service interfaces for developers.

Username	RHT_OCP4_DEV_USER	boyerje-us
Password	RHT_OCP4_DEV_PASSWORD	<>
API Endpoint	RHT_OCP4_MASTER_API	https://api.ocp-na2.prod.nextcle.com:6443
Console Web Application		https://console-openshift-console.apps.ocp-na2.prod.nextcle.com
Cluster ID ...

Container images are named based on the following syntax: `registry_name/user_name/image_name:tag`

Example of images used:

```shell
# login to a registry
sudo podman login -u username -p password registry.access.redhat.com
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

**Quay.io** introduces several features, such as server-side image building, fine-grained access controls, and automatic scanning of images for known vulnerabilities.

To configure registries for the podman command, you need to update the `/etc/containers/registries.conf`. The podman search command finds images from all the registries listed in this file.

```
[registries.search]
registries = ["registry.access.redhat.com", "quay.io"]
```

Use an FQDN and port number (5000 default) to identify a registry. 

By default, Podman stores container images in the /var/lib/containers/storage/overlay-images directory.

Existing images from the Podman local storage can be saved to a .tar file using the `podman save` command.

```shell

# Retrieve the list of external files and directories that Podman mounts to the running container
sudo podman inspect -f "{{range .Mounts}}{{println .Destination}}{{end}}" official-httpd
# list of modified files in the container file system
sudo podman diff official-httpd
# Commit the changes to a new container image with a new name
sudo podman commit -a 'Jerome' official-httpd do180-custom-httpd
sudo podman save [-o FILE_NAME] IMAGE_NAME[:TAG]
sudo podman tag quay.io/jbcodeforce/do180-custom-httpd:v1.0
sudo podman push quay.io/jbcodeforce/do180-custom-httpd:v1.0

sudo podman build -t NAME:TAG DIR
```

Red Hat Software Collections Library  is the source of most container images

### Deploying Containerized Applications on OpenShift

```shell
# login to cluster 
oc login -u ${RHT_OCP4_DEV_USER} -p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
# Create a new project named "youruser-ocp"
oc new-project ${RHT_OCP4_DEV_USER}-ocp
# Create a temperature converter application written in PHP using the php:7.1 image stream tag. The source code is in the Git repository at https://github.com/RedHatTraining/DO180-apps/
c new-app php:7.1~https://github.com/RedHatTraining/DO180-apps --context-dir temps --name temps
# Monitor progress of the build
oc logs -f bc/temps
# Verify that the application is deployed.
oc get pods -w
# Expose the temps service to create an external route for the application.
oc expose  svc/temps
# Determine route URL
 oc get route/temps
```

## Compendium

* [Open Container initiave](https://www.opencontainers.org/)
* [Podman, the daemonless container engine for developing, managing, and running OCI Containers on your Linux System](https://podman.io/)
* [Quay image registry](https://quay.io)
* [kubernetes](https://kubernetes.io/)
* [openshift](https://www.openshift.com/)
* [Git commands summary](https://rol.redhat.com/rol/app/apd.html)