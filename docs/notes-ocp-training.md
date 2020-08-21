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

# examine the content of the environment variable of a container
sudo podman exec todoapi env
```

Red Hat Software Collections Library  is the source of most container images

### Deploying Containerized Applications on OpenShift

Lab:

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

### Deploy multi-container app

Podman uses Container Network Interface (CNI) to create a software-defined network (SDN) between all containers in the host. Unless stated otherwise, CNI assigns a new IP address to a container when it starts.

Each container exposes all ports to other containers in the same SDN. As such, services are readily accessible within the same network. The containers expose ports to external networks only by explicit configuration.

Using environment variables allows you to share information between containers with Podman. However, there are still some limitations and some manual work involved in ensuring that all environment variables stay in sync, especially when working with many containers

Any service defined on Kubernetes generates environment variables for the IP address and port number where the service is available. Kubernetes automatically injects these environment variables into the containers from pods in the same namespace

Get the list of templates from where to create k8s resources like Secret, a Service, a PersistentVolumeClaim, and a DeploymentConfig: ` oc get template -n openshift` and then from one of the template: `oc get template mysql-persistent -n openshift -o yaml`.

With template, you can publish a new template to the OpenShift cluster so that other developers can build an application from the template.

Get the parameters of a template: ` process --parameters mysql-persistent -n openshift`.

From the template create the app resources file and then deploy the app:

```shell
# first export the template
oc get template mysql-persistent -o yaml -n openshift > mysql-persistent-template.yaml
# identify appropriate values for the template parameters and process the template
oc process -f mysql-persistent-template.yaml -p MYSQL_USER=dev -p MYSQL_PASSWORD=$P4SSD -p MYSQL_DATABASE=bank -p VOLUME_CAPACITY=10Gi | oc create -f -
# or using new-app
oc new-app --template=mysql-persistent -p MYSQL_USER=dev -p MYSQL_PASSWORD=$P4SSD -p MYSQL_DATABASE=bank -p VOLUME_CAPACITY=10Gi
```

Lab:

```shell
# login to cluster 
oc login -u ${RHT_OCP4_DEV_USER} -p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
# Create a new project named "youruser-"
oc new-project ${RHT_OCP4_DEV_USER}-deploy
# Build the MySQL Database image
sudo podman build -t do180-mysql-57-rhel7 .
# Push the MySQL image to the your Quay.io repository.
sudo podman login quay.io -u ${RHT_OCP4_QUAY_USER}
sudo podman tag do180-mysql-57-rhel7 quay.io/${RHT_OCP4_QUAY_USER}/do180-mysql-57-rhel7
sudo podman push quay.io/${RHT_OCP4_QUAY_USER}/do180-mysql-57-rhel7
```
## Compendium

* [Open Container initiave](https://www.opencontainers.org/)
* [Podman, the daemonless container engine for developing, managing, and running OCI Containers on your Linux System](https://podman.io/)
* [Quay image registry](https://quay.io)
* [kubernetes](https://kubernetes.io/)
* [openshift](https://www.openshift.com/)
* [Git commands summary](https://rol.redhat.com/rol/app/apd.html)