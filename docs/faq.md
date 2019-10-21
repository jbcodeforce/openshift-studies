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

You can try [code-ready]() too which is the last release on how to have a single master and single worker.


## Login and push image to private registry

Openshift manages its own image private registry service. The default name and URL is docker-registry-default.apps.green-with-envy.ocp.csplab.local. Below is the step to push a docker images

* Login to openshift cluster

```
oc login --username john --password password --server=https://master01.green-with-envy.ocp.csplab.local
```

* Login to docker registry:

```
docker login -u john -p $(oc whoami -t) docker-registry-default.apps.green-with-envy.ocp.csplab.local
```

If you get this message: `Error response from daemon: Get https://docker-registry-default.apps.green-with-envy.ocp.csplab.local/v2/: x509: certificate signed by unknown authority`, add the certificate to the docker client certificates:

        * Get the certificate: `oc extract -n default secrets/registry-certificates --keys=registry.crt`
        * Put the certificate in `~/.docker/certs.d/docker-registry-default.apps.green-with-envy.ocp.csplab.local` 
        * Restart docker desktop

* Tag the image with registry name:

```
docker tag ibmcase/kc-ordercommandms docker-registry-default.apps.green-with-envy.ocp.csplab.local/reefershipmentsolution/kc-ordercommandms
```

* Push the image

```
docker push docker-registry-default.apps.green-with-envy.ocp.csplab.local/reefershipmentsolution/kc-ordercommandms
```

* Accessing the registry console: https://registry-console-default.apps.green-with-envy.ocp.csplab.local/

* Generate deployment.yaml and services.yaml from helm templates:

```
helm template --set image.repository=docker-registry.default.svc:5000/reefershipmentsolution/kc-ordercommandms --set kafka.brokersConfigMap=kafka-brokers --set eventstreams.enabled=true --set eventstreams.apikeyConfigMap=eventstreams-apikey --set serviceAccountName=kcontainer-runtime  --namespace reefershipmentsolution --output-dir templates chart/ordercommandms
```

* Refresh an existing pod with the new image

```
oc delete -f templates/ordercommandms/templates/
oc apply -f templates/ordercommandms/templates/
```

## Deployment

### Deploy any docker image

```
oc new-app busybox
```