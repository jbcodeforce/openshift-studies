# Appsody Stack usage and new one

[Appsody](https://appsody.dev/) provides pre-configured container images (stacks) and project templates for a growing set of popular open source runtimes and frameworks, providing a foundation on which to build applications for Kubernetes and Knative deployments.

To install **appsody** on mac with home brew:   `brew install appsody/appsody/appsody`. To update to new version: `brew upgrade appsody`

## How it works

Appsody includes a CLI and a daemon to control the life cycle of the application. A developer uses the CLI to create a new application. The application contains a Docker image created from a Appsody stack and injects the template defines in the stack with the application developer’s files into it. The daemon watch change to files and build and deploy continuously.
Stacks are defined in a repository. Repositories can be referenced from remote sources (e.g., GitHub) or can be stored locally on a filesystem.

Appsody helps developer to do not worry about the details of k8s deployment and build. During a Appsody run, debug or test step, Appsody creates a Docker container based on the parent stack Dockerfile, and combines application code with the source code in the template. 

Stack has one docker file to help building the application and control the build, run and test steps of appsody. It also includes a second Dockerfile in the images/project folder to deockerize the final app. The Docker file is responsible for ensuring the combined dependencies are installed in the final image.

When designing a stack, we need to decide who control the application: a web server in which the developer, user of the stack, is adding new end points, or the developer controlling how the app starts and runs.


See details in [this note](https://developer.ibm.com/technologies/containers/tutorials/create-appsody-stack).

To get appsody environment variables description in the [product documentation](https://appsody.dev/docs/stacks/environment-variables)

## Summary of common appsody CLI commands

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

When a source code project is initialized with Appsody, you get a local Appsody development container where you can do:

```
appsody run 
appsody test 
appsody debug 
```

* Build: You can use the `appsody build` command to generate a deployment Docker image on your local Docker registry, and then manually deploy that image to your runtime platform of choice.
* You can use the `appsody deploy` command to deploy the same deployment Docker image directly to a Kubernetes cluster that you are using for testing or staging.
* You can delegate the build and deployment steps to an external pipeline, such as a Tekton pipeline that consumes the source code of your Appsody project after you push it to a GitHub repository.


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