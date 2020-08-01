# Code Ready

Red Hat tools to help developer get the most of k8s and get started quickly. It includes workspace, container, studio, builder, toolchain and dependencies.

## ODO: Openshift Do

[ODO](https://docs.openshift.com/container-platform/4.3/cli_reference/openshift_developer_cli/understanding-odo.html) is a CLI for developer to abstract kubernetes. [odo](https://www.katacoda.com/openshift/courses/introduction/developing-with-odo) can build and deploy your code to your cluster immediately after you save your changes. Odo abstracts away Kubernetes and OpenShift concepts.

`odo` helps manage the components in a grouping to support the application features. A selection of runtimes, frameworks, and other components are available on an OpenShift cluster for building your applications. This list is referred to as the Developer Catalog.

Installing odo:

```shell
curl -L https://mirror.openshift.com/pub/openshift-v4/clients/odo/latest/odo-darwin-amd64 -o /usr/local/bin/odo
chmod +x /usr/local/bin/odo
```

List existing software runtime catalog deployed on a cluster (For example Java and nodejs are supported runtimes):

```shell
odo catalog list components
```

To create a component (create a config.yml) from a java springboot app, once the jar is built, the following command defines (a `backend` named component) to run it on top of the java runtime:

```shell
odo create java backend --binary target/wildwest-1.0.jar
```
The component is not yet deployed on OpenShift. With an odo create command, a configuration file called config.yaml has been created in the local directory. To see the config use:

```shell
odo config view

COMPONENT SETTINGS
------------------------------------------------
PARAMETER         CURRENT_VALUE
Type              java:8
Application       app
Project           myproject
SourceType        binary
Ref
SourceLocation    target/wildwest-1.0.jar
Ports             8080/TCP,8443/TCP,8778/TCP
Name              backend
```

Then to deploy the binary jar file to Openshift:

```shell
odo push
```
OpenShift has created a container to host the backend component, deployed the container into a pod running on the OpenShift cluster, and started up the backend component.
You can view the backend component being started up, in the `Developer` perspective, under the `Topology` view. When a dark blue circle appears around the backend component, the pod is ready and the backend component container will start running on it. (A light blue ring means the pod is in a pending state and hasn't started yet)

OpenShift provides mechanisms to publish communication bindings from a program to its clients. This is referred to as linking. To link the current frontend component to the backend: 

```shell
odo link backend --component frontend --port 8080
```

This will inject configuration information into the frontend about the backend and then restart the frontend component.

To expose an application to external client, we need to add a URL:

```shell
odo url create frontend --port 8080
odo push
```

To adapt to the developer changes, we can tell odo to watch for changes on the file system in the background using:

```
odo watch
```

Once the change is recognized, odo will push the changes to the frontend component and print its status to the terminal. 

See [odo github](https://github.com/openshift/odo)


## Code Ready Container

Red Hat CodeReady Containers brings a minimal OpenShift 4.0 or newer cluster to your local computer. See getting started [here](https://code-ready.github.io/crc/)

### Install quick summary

* Download 1.8 G binary from [here](https://cloud.redhat.com/openshift/install/crc/installer-provisioned), extract it in a folder with $PATH. It shoud just create a crc command.
* Run `crc setup` to set up the environment of your host machine 
* Start the VM `crc start`. Keep the password for the developer user.
* `oc login -u developer -p developer https://api.crc.testing:6443` 
* Stop the VM `crc stop`
* Access the console `crc console` or `crc console --credentials`
* Delete the VM: `crc delete`
* The `crc ip` command can be used to obtain the VM IP address as needed

!!! Notes
    CodeReady Containers creates a /etc/resolver/testing file which instructs macOS to forward all DNS requests for the testing domain to the CodeReady Containers virtual machine.CodeReady Containers also adds an api.crc.testing entry to /etc/hosts pointing at the VM IP address. This is needed by the oc binary.


To access the OpenShift web console, follow these steps:

1. Run crc console. This will open your web browser and direct it to the web console.
2. Log in to the OpenShift web console as the developer user with the password printed in the output of the crc start command or by running: `crc console --credentials`

To access to the administrator user login as kubeadmin, something like:

`oc login -u kubeadmin -p 7z6T5-qmTth-oxaoD-p3xQF https://api.crc.testing:6443`

### Use oc with CRC

To access the OpenShift cluster via the oc command:

* `crc oc-env`
* Get the Cluster Operators: `oc get co`

## CodeReady workspaces

CodeReady is a web-based IDE running on Openshift and in a web browser, it is based on Eclipse Che 7.

It is installed using the OperatorHub Catalog present in the OpenShift web console, [see installation note](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/2.1/html/installation_guide/index)

It uses the concept of devfile to define the development environment as portable and committable to github.

