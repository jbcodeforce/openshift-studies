# Code Ready

Red Hat tools to help developer get the most of k8s and get started quickly. It includes workspace, container, studio, builder, toolchain and dependencies.


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

