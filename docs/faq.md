# OpenShift FAQ

### Is it possible to run openshift in docker for development?

Yes using the [oc cluster up](https://github.com/openshift/origin/blob/master/docs/cluster_up_down.md) to download the openshift/origin-control-plane:v3.11 and start it. 
If you get the following error
> Checking if insecured registry is configured properly in Docker ...
error: did not detect an --insecure-registry argument on the Docker daemon

Add the `"insecure-registries": ["172.30.0.0/16"],` to the Daemon properties:


