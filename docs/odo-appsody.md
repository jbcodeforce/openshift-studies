# ODO and Appsody Summary

## Openshift DO

[ODO CLI creating applications on OpenShift](https://docs.openshift.com/container-platform/4.2/cli_reference/openshift_developer_cli/understanding-odo.html).

The main value propositions are:

* Abstracts away complex Kubernetes and OpenShift.
* Detects changes to local code and deploys it to the cluster automatically, giving instant feedback to validate changes in real time
* Leverage source 2 images to produce ready-to-run images by building source code without the need of a Dockerfile

The [basic commands](https://docs.openshift.com/container-platform/4.3/cli_reference/openshift_developer_cli/odo-cli-reference.html#basic-odo-cli-commands_odo-cli-reference):

```shell
# login to a ROKS
odo  login --token=s....vA c100-e.us-south.containers.cloud.ibm.com:30040
# Create a new project in OCP
odo project create jbsandbox
# change project inside OCP
odo project set jbsandbox 
# Create a component (from an existing project for ex)... then following the question
odo component create
# list available component
odo catalog list components
# push to ocp
odo push
# delete an app
odo app list
odo app delete myapp
```

Push a component creates 2 pods: one ...app-deploy and one ...-app

### Important concepts

See details [in odo architecture section](https://docs.openshift.com/container-platform/4.2/cli_reference/openshift_developer_cli/odo-architecture.html).

* **Init containers** are specialized containers that run before the application container starts and configure the necessary environment for the application containers to run. Init containers can have files that application images do not have, for example setup scripts
* **Application** container is the main container inside of which the user-source code executes. It uses two volumes: emptyDir and PersistentVolume. The data on the PersistentVolume persists across Pod restarts.
* odo creates a **Service** for every application Pod to make it accessible for communication
* odo push workflow: 
    * create resources like deployment config, service, secrets, PVC
    * index source code files 
    * push code into the application container
    * execute assemble and restart.

The figure below illustrates those components:

![](./images/odo-elements.png)

## Appsody

[Appsody](https://appsody.dev/) provides pre-configured container images (stacks) and project templates for a growing set of popular open source runtimes and frameworks, providing a foundation on which to build applications for Kubernetes and Knative deployments.

To install **appsody** on mac with home brew:   `brew install appsody/appsody/appsody`. To update to new version: `brew upgrade appsody`

## How it works

Appsody includes a CLI and a daemon to control the life cycle of the application. Developers use the CLI to create a new application from an existing "stack" (1).

![Appsody components](images/appsody-concept.png)

Stacks are defined in a repository. Repositories can be referenced from remote sources (e.g., GitHub) or can be stored locally on a filesystem.

First we can get the list of templates available via the command:

```shell
appsody list
```

Then create our own application using one of the template like:

```shell
mkdir projectname
appsody init java-microprofile
```

In general the command is `appsody init <repository-name>/<stack>`. It is possible to initialize an existing project using the command: `appsody init <stackname> --no-template`.

Appsody helps developer to do not worry about the details of k8s deployment and build. During a Appsody run, debug or test step (2), Appsody creates a Docker container based on the parent stack Dockerfile, and combines application code with the source code in the template.

When a source code project is initialized with Appsody, you get a local Appsody development container where you can do the following commmands:

```shell
appsody run
appsody test
appsody debug
```

One of the above command creates a daemon which monitors changes to any files and build and start a new docker container continuously.

```shell
# The daemon
ps -ef | grep appsody
501 4156 93070 appsody run
# the docker container
501 56789 4156 docker run --rm -p 7777:7777 -p 9080:9080 -p 9443:9443 --name scoring-mp-dev -v /Users/jeromeboyer/.m2/repository:/mvn/repository -v /Users/jeromeboyer/Code/jbcodeforce/myEDA/refarch-reefer-ml/scoring-mp/src:/project/user-app/src -v /Users/jeromeboyer/Code/jbcodeforce/myEDA/refarch-reefer-ml/scoring-mp/pom.xml:/project/user-app/pom.xml -v appsody-controller-0.3.3:/.appsody -t --entrypoint /.appsody/appsody-controller docker.io/appsody/java-microprofile:0.2 --mode=run
```

The other basic commands are:

* **Build**: You can use the `appsody build` command to generate a deployment Docker image on your local Docker registry, and then manually deploy that image to your runtime platform of choice.
* You can use the `appsody deploy` command (3) to deploy the same deployment Docker image directly to a Kubernetes cluster that you are using for testing or staging. See next section.
* You can delegate the build and deployment steps to an external pipeline, such as a Tekton pipeline that consumes the source code of your Appsody project after you push it to a GitHub repository.
* `appsody stop`

See [Appsody CLI](https://appsody.dev/docs/using-appsody/cli-commands/) commands.

### Create a python flask app

The stack is not for production and is not fully supported. Here is an example of creating a simple webapp with flask, flask cli and gunicorn

```shell
# Get the default template from the stack
appsody init incubator/python-flask
# build an image with a name = the folder name based on the dockerfile from the stack
appsody build
# or run it directly
appsody run
```

You can add your own dockerfile to extend existing one. With `docker images` you can see what `appsody build` created, then you can use this image as source for your own docker image

```dockerfile
FROM <nameofyourapp>
ADD stuff
CMD change the command
```

To add your own code.

### Create a springboot app

```shell
appsody init java-microprofile
```

### Create a microprofile 3.0 app

```shell
appsody init java-microprofile
```

### Deployment

Once logged to a k8s cluster, a `appsody deploy -t dockerhub/imagename --push -n yournamespace` (3) will do the following:

* deploy appsody operator into the given namespace if no operator found
* call `appsody build` and create a deployment image with the given tag
* push the image to docker hub or other repository
* create the `app-deploy.yaml` deployment manifest
* Apply it with:  `kubectl apply -f app-deploy.yaml`  within the given namespace

Verify the operator is deployed:

```shell
oc get pods
NAME                               READY     STATUS             RESTARTS   AGE
appsody-operator-d8dfb4f5f-4dpwk   1/1       Running            0          4m34s
```

As part of the deployment manifest a service and a route are created. For example using a microprofile app the following command will verify everthing went well.

```shell
curl http://scoring-mp-eda-sandbox.apps.green.ocp.csplab.local/health
```

```shell
appsody deploy delete -n yournamespace
```

To ensure that the latest version of your app is pushed to the cluster, use the -t flag to add a unique tag every time you redeploy your app. Kubernetes then detects a change in the deployment manifest, and pushes your app to the cluster again.

## Defining your own stack

Stack has one dockerfile to help building the application and control the build, run and test steps of appsody. It also includes a second Dockerfile in the images/project folder to "dockerize" the final app. The Dockerfile is responsible for ensuring the combined dependencies are installed in the final image.

When designing a stack, we need to decide who control the application: a web server in which the developer, user of the stack, is adding new end points, or the developer is controlling how the app starts and runs.

See details in [this note](https://developer.ibm.com/technologies/containers/tutorials/create-appsody-stack).

To get appsody environment variables description in the [product documentation](https://appsody.dev/docs/stacks/environment-variables)

See [this other tutorial here](https://developer.ibm.com/tutorials/create-appsody-stack/).

Some considerations to address:

* select the technologies and libraries to use
* how to verify dependencies
* what kind of sample starter application
* How to enable adding new libraries
* What docker image repository to use, and what credentials
* Install Custom resource. 

Here is a summary of the steps to create a Kafka java stack for consumer and producer:

* Look to local cache for appsody stack

```shell
$ export APPSODY_PULL_POLICY=IFNOTPRESENT
```

* Create a starter stack, as a scaffold. 

```shell
$ appsody stack create gse-eda-java-stack --copy incubator/java-microprofile
```

* build the stack using the `Dockerfile-stack` file

```shell
$ appsody stack package


Your local stack is available as part of repo dev.local
```

* Test your stack scaffold

```shell
$ appsody init dev.local/gse-eda-java-stack

 Successfully initialized Appsody project with the dev.local/gse-eda-java-stack stack and the default template.
```

* Start the application scafold using `appsody run`

* Modify the `Dockerfile-stack` file to include the base image and dependencies for the server and other predefined code.

## Future readings

* [Introduction to Appsody: Developing containerized applications for the cloud just got easier](https://developer.ibm.com/blogs/introduction-to-appsody/)
* [Video Appsody overview](https://developer.ibm.com/blogs/introduction-to-appsody/)
* [Kabanero - Appsody - Tekton - Openshift](https://github.ibm.com/steve-arnold/kabanerodemoguide/blob/master/README.md)