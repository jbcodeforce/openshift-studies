# OpenShift FAQ

## Access

### How to get a token for login

As soon as a service account is created, two secrets are automatically added to it:

* an API token
* credentials for the OpenShift Container Registry

To access the token you can go to the Openshift container platform web console, and select the user icon on the top right and the command: `copy login command` you should have the token.

```
oc login -u apikey -p I173tzup --server=https://c2-e.us-east.containers.cloud.ibm.com:21070
```

### Strange message from login

Some oc login command may return a strange message: `error: invalid character '<' looking for beginning of value`. This is due to the fact that the response is a HTML page. This is a problem of server URL. The Server parameter has to correspond to your OpenShift API server endpoint.
When deploy on premise be sure to use the k8s master URL. 

### Is it possible to run openshift in docker for development?

Yes using the [oc cluster up](https://github.com/openshift/origin/blob/master/docs/cluster_up_down.md) to download the openshift/origin-control-plane:v3.11 and start it. 
If you get the following error
> Checking if insecured registry is configured properly in Docker ...
error: did not detect an --insecure-registry argument on the Docker daemon

Add the `"insecure-registries": ["172.30.0.0/16"],` to the Daemon properties:

##

