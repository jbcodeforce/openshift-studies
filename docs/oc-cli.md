# oc cheat sheet 

oc is a tool written in Go to interact with an openshifr cluster (one at a time). State about the current login session is stored in the home directory of the local user running the oc command.

#### Login

```
oc login --username collaborator --password collaborator 
```

Log in to your server using a token for an existing session.

```
oc login --token <token>
```

Get the list of projects

```
oc get projects
```

To create a project, that is mapped to a kubernetes namespace:
```
$ oc new-project <project_name> --description="<description>" --display-name="<display_name>"
```

To assess who you are and get login userid:
```
oc whoami
```

See what the current context: 
```
oc whoami --show-context
```

verify which server you are logged into

```
oc whoami --show-server
```


See cluster:

```
oc config get-clusters
```

Show a list of contexts for all sessions ever created. For each context listed, this will include details about the project, server and name of user, in that order
```
oc config get-contexts
```

#### Add access for a user to one of your project

```
oc adm policy add-role-to-user view developer -n myproject
> role "view" added: "developer"
```

The role could be `edit | view | admin`

#### Project commands

Select a project once logged to openshift:
```
oc project <projectname>
```

Search a docker image and see if it is valid:

```
oc new-app --search openshiftkatacoda/blog-django-py
```

Deploy an image to a project and link it to github:
```
oc new-app --docker-image=<docker-image> [--code=<source>]
```

The image will be pulled down and stored within the internal OpenShift image registry. You can list what image stream resources have been created within a project by running the command:

```
oc get imagestream -o name
``` 

To expose the application created so it is available outside of the OpenShift cluster, you can run the command:
```
oc expose service/blog-django-py
```

To view the hostname assigned to the route created from the command line, you can run the command:

```
oc get route/blog-django-py
```

To see a list of all the resources that have been created in the project
```
oc get all -o name
```

Or by using a label selector: 
```
oc get all --selector app=blog-django-py -o name
```

Get detail of a resource:
```
oc describe route/blog-django-py
```

Deleting an application and all its resources, using label:
```
oc delete all --selector app=blog-django-py
```

To import an image without creating a container: 
```
oc import-image openshiftkatacoda/blog-django-py --confirm
```

Then to deploy an instance of the application from the image stream which has been created, run the command:
```
oc new-app blog-django-py --name blog-1
```

List what image stream resources have been created within a project by running the command:

```
oc get imagestream -o name
```

Create an app from the source code, and use source to image build process to deploy the app:

```
oc new-app python:latest~https://github.com/jbcodeforce/order-producer-python -name appname
```

Then to track the deployment progress:
```
oc logs -f bc/<appname>
```
The dependencies are loaded, the build is scheduled and executed, the image is uploaded to the registry, and started. See the workflow in the diagram below.

![](s2i-workflow.png)

The first time the application is deployed, if you want to expose it to internet do:
```
oc expose service/<appname>
```

When using source to image approach, it is possible to trigger the build via:

```
oc start-build app-name
```

You can use oc logs to monitor the log output as the build runs. You can also monitor the progress of any builds in a project by running the command:

```
oc get builds --watch
```

To trigger a binary build, without committing to github, you can use the local folder to provide the source code:

```
oc start-build app-name --from-dir=. --wait
```

*The --wait option is supplied to indicate that the command should only return when the build has completed.*

Switch back to minishift context:

```
oc config use-context minishift
```




