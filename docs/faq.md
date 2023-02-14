# OpenShift FAQ

## Access

To get connection details from the config map in the kube-public namespace:

```
kubectl get cm ibmcloud-cluster-info -n kube-public -o yaml
```

The cluster_address value for the master address, and the cluster_router_https_port for the port number.

### How to get a token for login

As soon as a service account is created, two secrets are automatically added to it:

* an API token
* credentials for the OpenShift Container Registry

To access the token you can go to the Openshift container platform web console, and select the user icon on the top right and the command: `copy login command` you should have the token.

```
oc login -u apikey -p I173tzup --server=https://c2-e.us-east.containers.cloud.ibm.com:21070
```

### Strange message from login

Some `oc login` command may return a strange message: `error: invalid character '<' looking for beginning of value`. 
This is due to the fact that the response is a HTML page. This is a problem of server URL. The Server parameter has to correspond 
to your OpenShift API server endpoint.
When deploy on premise be sure to use the k8s master URL.

## Login and push image to private registry

OpenShift could manage its own image private registry service. The default name and URL is `docker-registry-default.apps....`. 
See the product [documentation here](https://docs.openshift.com/container-platform/3.9/install_config/registry/deploy_registry_existing_clusters.html#registry-non-production-use) to install it.

 Below is the step to push a docker images

* Login to OpenShift cluster (get secure token from OpenShift console)
* If not done before add registry-viewer role to your user: `oc policy add-role-to-user registry-viewer $(oc whoami)` 
and `oc policy add-role-to-user registry-editor $(oc whoami)`
* Look up the internal OpenShift Docker registry address by using the following command:

```
kubectl get routes docker-registry -n default
```

* Login to docker registry:

```sh
docker login -u john -p $(oc whoami -t) docker-registry-default.apps.green-with-envy.ocp.csplab.local
```

If you get this message: `Error response from daemon: Get https://docker-registry-default.apps.green-with-envy.ocp.csplab.local/v2/: x509: certificate signed by unknown authority`, 
add the certificate to the docker client certificates:

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

* Refresh an existing pod with the new image using `oc delete <deployment.yaml>` and `oc apply <deployment.yaml>`

## Deployment

### Deploy any docker image

Just reference the docker image name from the dockerhub public repository

```sh
oc new-app busybox
```

For mongodb using a local env file to specify the different environment variables to be used for deployment

```sh
oc new-app --env-file=mongo.env --docker-image=openshift/mongodb-24-centos7
```

## Copy a file to an existing running container

```sh
# os rsync local folder to running pod
oc rsync $(pwd) my-connect-connect-54485b7896-k5lsj:/tmp
oc rsh my-connect-connect-54485b7896-k5lsj 
ls /tmp
# can copy file too
```

## How to setup TLS/SSL certificate

The approach is to use secret and mounted volume to inject the SSL certifcate file so the app can use it to connect over TLS.

If you have the key and certificates as remoteapptoaccess.key and remoteappaccess.crt, you may need to encode them with base64:

```shell
$ base64 remoteapptoaccess.keys
LS0934345DE....
$ base64 remoteapptoaccess.crt
SUPERSECRETLONGSTRINGINBASE64FORMAT
```

Then create a TLS secret descriptor for kubernetes:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: remoteapp-tls-secret
type: Opaque
data:
  remoteapptoaccess.key: 
  LS0934345DE...
  remoteapptoaccess.crt:
  SUPERSERCRETLONGSTRINGINBASE64FORMAT
```

If you only have the crt file, you just define the data for it.

In the client app deployment.yaml set a mount point: (the following deployment example, is not complete, there are missing arguments linked to the app itself)

```yaml
apiVersion:  apps/v1
kind: Deployment 
metadata:
  labels:
    app: clientApp
  name: clientApp
spec:
  replicas: 1
  spec:
      containers:
      - image: yourregistry/yournamespace/imagename
        name: clientapp
        volumeMounts:
          - mountPath: "/client/path/inside/container/ssl"
            name: ssl-path
            readOnly: true
        ports:
        - containerPort: 80
      volumes:
        - name: ssl-path
          secret:
            secretName: remoteapp-tls-secret
```

This declaration will add two files (remoteapptoaccess.key, remoteapptoaccess.crt) under the `/client/path/inside/container/ssl` folder.
If the SSL certs and keys are not in the default folder expected by the application, environment variables should specify the paths.

## What's new in OpeShift 4.6

* installer: support disconnected env.

Core:

* remote worker nodes: need to be in the same subnetwork. Share the control plane / supervisor. Tolerant to disruption.  
* Full stack automation (Installer Provisioned Infrastructure) installation on bare metal
* serverless eventing
* kubernetes 1.19
* Open Virtual Network (OVN): CNI network plugin
* security compliance operator.
* monitoring your own services. 
* new log forwarding API (ClusterLogForwarder CRD): to elastic search, kafka, fluentd, syslog...

## Project removal stay in Terminating state

See [this note]().

* List resources not deleted: example on the project edademo-dev

```sh
 oc api-resources --verbs=list --namespaced -o name | xargs -n 1 oc get --show-kind --ignore-not-found -n edademo-dev
```

* Remove for each object still present, the finalizers declaration:

```sh
# Example for an argocd app which has created the project
oc patch -n edademo-dev rolebinding/edademo-dev-rolebinding --type=merge -p '{"metadata": {"finalizers":null}}'
oc patch -n edademo-dev rolebinding/argocd-admin   --type=merge -p '{"metadata": {"finalizers":null}}'
oc patch -n edademo-dev rolebinding/edit  --type=merge -p '{"metadata": {"finalizers":null}}'
```