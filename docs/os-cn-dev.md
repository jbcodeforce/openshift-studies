# Cloud Native Development Summary

## Source to image (s2i)

Source to image toolkit aims to simplify the deployment to OpenShift. It uses a build image to execute an assembly script that builds code and docker image without Dockerfile.  

The following figure, shows the resources created by the `oc new-app` command when the argument is an application source code repository.

 ![s2i](https://rol.redhat.com/rol/static/static_file_cache/do180-4.2/OpenShift-dc-and-bc-3.png)

From an existing repository, `s2i create` add a set of elements to define the workflow into the repo. For example the command below will add Dockerfile and scripts to create a build image named `ibmcase/buildorderproducer` from the local folder where the code is.

```
s2i create ibmcase/buildorderproducer .
```

When the assemble script is done, the container image is committed to internal image repository. The CMD part of the dockerfile execute a run script.

Here is another command to build the output image using existing build image on local code:

```
s2i build --copy .  centos/python-36-centos7 ibmcase/orderproducer
```

!!! Note
    s2i takes the code from git, so to use the local code before committing it to github, add the `--copy` argument.

* OpenShift builds applications against an image stream. The OpenShift installer populates several image streams by default during installation.  

```shell
 oc get is -n OpenShift
```

If only a source repository is specified, oc new-app tries to identify the correct image stream to use for building the application

## ODO: Openshift Do


[ODO](https://odo.dev/docs/understanding-odo/) is a CLI for developer to abstract kubernetes. It can build and deploy your code to your cluster immediately after you save your changes. `odo` helps manage the components in a grouping to support the application features. A selection of runtimes, frameworks, and other components are available on an OpenShift cluster for building your applications. This list is referred to as the Developer Catalog.

The main value propositions are:

* Abstracts away complex Kubernetes and OpenShift commands and configurations.
* Detects changes to local code and deploys it to the cluster automatically, giving instant feedback to validate changes in real time

V2.0 is merging with Appsody where stacks are [devfile in odo](https://odo.dev/docs/deploying-a-devfile-using-odo/). A **devfile** is a portable file that describes your development environment. See some [devfile examples](https://github.com/odo-devfiles)

### Important concepts

* **Init containers** are specialized containers that run before the application container starts and configure the necessary environment for the application containers to run. Init containers can have files that application images do not have, for example setup scripts
* **Application** container is the main container inside of which the user-source code executes. It uses two volumes: emptyDir and PersistentVolume. The data on the PersistentVolume persists across Pod restarts.
* odo creates a **Service** for every application Pod to make it accessible for communication
* odo push workflow: 
    * create resources like deployment config, service, secrets, PVC
    * index source code files 
    * push code into the application container
    * execute assemble and restart.


### Installing odo

[Last instructions to install](https://odo.dev/docs/installing-odo/). 

```shell
# For MAC
curl -L https://mirror.openshift.com/pub/openshift-v4/clients/odo/latest/odo-darwin-amd64 -o /usr/local/bin/odo
chmod +x /usr/local/bin/odo
# For VSCode: Command P and
ext install redhat.vscode-openshift-connector
```

List existing software runtime catalog deployed on a cluster (For example Java and nodejs are supported runtimes):

```shell
odo catalog list components
```

### Developing with ODO


```shell
# login to a OpenShift cluster like ROKS
odo  login --token=s....vA c100-e.us-south.containers.cloud.ibm.com:30040
# Create a new project in OCP
odo project create jbsandbox
# change project inside OCP
odo project set jbsandbox 
# Create a component (from an existing project for ex)... then follow the different questions
odo component create
# list available component, with v2, the devfile list is also returned
odo catalog list components
# push to ocp
odo push
# delete an app
odo app list
odo app delete myapp
```

Deploy a component to OpenShift creates 2 pods: one ...app-deploy and one ...-app

---
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
----




### Python Flask

Current base code is a Flask under [https://github.com/odo-devfiles/python-ex](https://github.com/odo-devfiles/python-ex)

```shell
mkdir project-name
# create the devfile and download starter code
odo create python --starter
# Deploy to OCP
odo push
```

## Kustomize and gitops

[Gitops](https://www.gitops.tech/)  is a way of implementing Continuous Deployment for cloud native applications.

The core idea of GitOps is having a Git repository that always contains declarative descriptions of the infrastructure currently desired in the production environment and an automated process to make the production environment match the described state in the repository.

There are at least two repositories: the **application** repository and the **environment configuration** repository.

There are two ways to implement the deployment strategy for GitOps: 

* **Push-based:** use CI/CD tools like jenkins, travis... to define a pipeline, triggered when application code source is updated, to build the container image and deploy the modified yaml files to the environment repo. Changes to the environment configuration repository trigger the deployment pipeline. It has to be used for running an automated provisioning of cloud infrastructure. See also [this tutorial](https://cloud.google.com/kubernetes-engine/docs/tutorials/gitops-cloud-build).
* **Pull-based:** An operator takes over the role of the pipeline by continuously comparing the desired state in the environment repository with the actual state in the deployed infrastructure.

A CICD based on git action will build the image and edit Kustomize patch to bump the expected container tag using the new docker image tag, then commit this changes to the gitops repo.

[Kustomize](https://kustomize.io/) is used to simplify the configuration of application and environment. Kustomize traverses a Kubernetes manifest to add, remove or update configuration options without forking. It is available both as a standalone binary and as a native feature of Kubectl. A [lot of examples here.](https://github.com/kubernetes-sigs/kustomize/tree/master/examples).

The simple way to organize the configuration is to use one `kustomize` folder, then one folder per component, then one overlay folder in which environment folder include `kustomization.yaml` file with patches.

```
└── postgres
    ├── base
    │   ├── configmap.yaml
    │   ├── kustomization.yaml
    │   ├── pvc.yaml
    │   ├── secret.yaml
    │   ├── service-account.yaml
    │   ├── statefulset.yaml
    │   ├── svc-headless.yaml
    │   └── svc.yaml
    ├── kustomization.yaml
    └── overlays
        └── dev
            ├── kustomization.yaml
            └── secret.yaml
```

Here is an example of `kustomization.yaml`:

```
bases:
  - ../../base
patchesStrategicMerge:
  - ./secret.yaml
```

The `patchesStrategicMerge` lists the resource configuration YAML files that you want to merge to the base kustomization. You must also add these files to the same repo as the kustomization file, such as `overlay/dev`. These resource configuration files can contain small changes that are merged to the base configuration files of the same name as a patch.


In GitOps, the pipeline does not finish with something like `oc apply..`. but it’s an external tool (Argo CD or Flux) that detects the drift in the Git repository and will run these commands.

Here is a command to install ArgoCD on k8s, see [details here](https://argoproj.github.io/argo-cd/getting_started/).

```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

For OpenShift:

* create an `argocd` project
* use operator hub, and install the operator in this project. Nothing to change in the default configuration
* deploy one instance of ArgoCD with the following customization:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: argocd
  namespace: argocd
spec:
  server:
    ingress:
      enabled: true
    route: 
      enabled: true
  dex:
    openShiftOAuth: true
    image: quay.io/ablock/dex
    version: openshift-connector
  rbac:
    policy: |
      g, system:cluster-admins, role:admin
```


An example of gitops with kustomize is in this [vaccine-gitops repo](https://github.com/ibm-cloud-architecture/vaccine-gitops)

## Future readings

* [GitOps - Operations by Pull Request](https://www.weave.works/blog/gitops-operations-by-pull-request)