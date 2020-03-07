# Deployments examples

## Deploy nodejs app using appsody

## Deploy zipkin from docker image

Install it, and expose it with a service

```
oc new-app --docker-image=openzipkin/zipkin

oc expose svc/zipkin
```
A new route is created visible with `oc get routes`. Once the hostname is added to a DNS or /etc/hosts. 

See zipkin architecture [article here](https://zipkin.io/pages/architecture.html)

## Deploy DB2


## Deploy sparks

[Using the operator, see this note](spark-on-os.md)

## Mongodb

Define an env file with the following variables:
MONGODB_USER=mongo
MONGODB_PASSWORD=mongo
MONGODB_DATABASE=reeferdb
MONGODB_ADMIN_PASSWORD=password

then run:

```
oc new-app --env-file=mongo.env --docker-image=openshift/mongodb-24-centos7
```

See the service:

```
oc describe svc mongodb-24-centos7
```

For more detail see [this note](https://docs.openshift.com/enterprise/3.0/using_images/db_images/mongodb.html)

## Deploy Jupyter lab

See [this note](https://blog.openshift.com/jupyter-openshift-part-2-using-jupyter-project-images/) to deploy Jupyter lab lastest image to Openshift using the Deploy Image choice. The deployment is done under the project `reefer-shipment-solution`

![](images/jupyterlab-1.png)


The environment variable needs to be set to get Jupyter lab. 

![](images/jupyterlab-2.png)

It takes multiple minutes to deploy. For the permission error due to the jovyan user not known, the command was:

```
oc adm policy add-scc-to-user anyuid -z default -n reefer-shipment-solution
```

```
 oc get routes
NAME  HOST/PORT   PATH      SERVICES                PORT       TERMINATION   WILDCARD
jupyterlab              jupyterlab-reefer-shipment-solution.greencluster-fa9ee67c9ab6a7791435450358e564cc-0001.us-east.containers.appdomain.cloud    all-spark-notebook   8888-tcp      None
```

Get the secuity token to login in via the pod logs.

```
oc get pods 
NAME                             READY     STATUS      RESTARTS   AGE
all-spark-notebook-2-z4dqx  
```

```
oc logs all-spark-notebook-2-z4dqx 
```

To avoid loosing the work on the notebook, we need to add PVC to `/home/jovyan/work` mount point

```
oc get dc

oc set volume dc/all-spark-notebook  --add --mount-path /home/jovyan/work --claim-size=1G
```
