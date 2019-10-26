# Code Ready container

Red Hat CodeReady Containers brings a minimal OpenShift 4.0 or newer cluster to your local computer. See getting started [here](https://code-ready.github.io/crc/)

## Install quick summary

* download 2.2 G binary, extract it in a folder with $PATH
* Run `crc setup` to set up the environment of your host machine 
* Start the VM `crc start` . Keep the password for the developer user.
* Stop the VM `crc stop`
* Access the console `crc console` or `crc console --credentials`
* Delete the VM: `crc delete`
* The `crc ip` command can be used to obtain the VM IP address as needed

## Use oc with CRC

To access the OpenShift cluster via the oc command:

* `crc oc-env`
* Run
* Log in by running the `oc login -u developer -p developer https://api.crc.testing:6443`