# oc cheat sheet 

oc is a tool written in Go to interact with an openshifr cluster (one at a time). State about the current login session is stored in the home directory of the local user running the oc command.

#### Login

```
oc login --username collaborator --password collaborator
```

Get the list of projects

```
oc get projects
```

```
oc whoami
```

See what the current context: 
```
oc whoami --show-context
```

See cluster:

```
oc config get-clusters
```

#### Add access for a user to one of your project

```
oc adm policy add-role-to-user view developer -n myproject
> role "view" added: "developer"
```


The role could be `edit | view | admin`

#### Project commands

Search an docker image and see if it is valid:
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

Get detail of a resouce:
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

Switch back to minishift context:

```
oc config use-context minishift
```




