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

## Deploy sparks

[Using the operator, see this note](spark-on-os.md)

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
