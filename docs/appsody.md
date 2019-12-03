# Appsody Stack usage and new one

[Appsody](https://appsody.dev/) provides pre-configured container images (stacks) and project templates for a growing set of popular open source runtimes and frameworks, providing a foundation on which to build applications for Kubernetes and Knative deployments.

To install **appsody** on mac with home brew:   `brew install appsody/appsody/appsody`. To update to new version: `brew upgrade appsody`

## How it works

Appsody includes a CLI and a daemon to control the life cycle of the application. A developer uses the CLI to create a new application. The application contains a Docker image created from a Appsody stack and injects the template defined in the stack with the application developer’s code into it. The daemon watch changes to any files and build and deploy continuously.
Stacks are defined in a repository. Repositories can be referenced from remote sources (e.g., GitHub) or can be stored locally on a filesystem.

Appsody helps developer to do not worry about the details of k8s deployment and build. During a Appsody run, debug or test step, Appsody creates a Docker container based on the parent stack Dockerfile, and combines application code with the source code in the template. 

Stack has one dockerfile to help building the application and control the build, run and test steps of appsody. It also includes a second Dockerfile in the images/project folder to "dockerize" the final app. The Dockerfile is responsible for ensuring the combined dependencies are installed in the final image.

When designing a stack, we need to decide who control the application: a web server in which the developer, user of the stack, is adding new end points, or the developer is controlling how the app starts and runs.


See details in [this note](https://developer.ibm.com/technologies/containers/tutorials/create-appsody-stack).

To get appsody environment variables description in the [product documentation](https://appsody.dev/docs/stacks/environment-variables)

## Summary of common appsody CLI commands

Add a repository, for example adding the kabanero repository:

```
appsody repo add kabanero https://github.com/kabanero-io/collections/releases/download/v0.1.2/kabanero-index.yaml 
```

To get the list of templates available: 

```
appsody list
```

To create a project from one of the template:

```
mkdir projectname
appsody init <repository-name>/<stack>
appsody init appsodyhub/java-microprofile
```

To initialize Appsody without using a template on the existing project:

```
appsody init <stack> --no-template 
```

When a source code project is initialized with Appsody, you get a local Appsody development container where you can do the following commmands:

```
appsody run 
appsody test 
appsody debug 
```

* **Build**: You can use the `appsody build` command to generate a deployment Docker image on your local Docker registry, and then manually deploy that image to your runtime platform of choice.
* You can use the `appsody deploy` command to deploy the same deployment Docker image directly to a Kubernetes cluster that you are using for testing or staging.
* You can delegate the build and deployment steps to an external pipeline, such as a Tekton pipeline that consumes the source code of your Appsody project after you push it to a GitHub repository.

## Create a python flask app

The stack is not for production and is not fully supported. Here is an example of creating a simple webapp with flask, flask cli and gunicorn

```
# Get the default template from the stack
appsody init incubator/python-flask
# build an image with a name = the folder name based on the dockerfile from the stack
appsody build
# or run it directly
appsody run
```

You can add your own dockerfile to extend existing one. With `docker images` you can see waht appsody build created, then you can use this image as source for your own docker image
```
FROM <nameofyourapp>
ADD stuff
CMD change the command
```

To add your own code 

## Create a springboot app

## Create a microprofile 3.0 app

## Create your own stack

See [the tutorial here](https://developer.ibm.com/tutorials/create-appsody-stack/) for detail.

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