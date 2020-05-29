# Deployments examples

Multiple ways to deploy an app to openshift:

1. Deploy an application from an existing Docker image. (Using `Deploy Image` in the project view.)

    ![1](images/deploy-image.png)

    !!! note
        There are two options: 

        * from an image imported in the openshift cluster, or built from a dockerfile inside the cluster. 
        * by accessing a remote image repository like `Dockerhub` or [quay.io](https://quay.io). The image will be pulled down and stored within the internal OpenShift image registry. The image will then be copied to any node in the OpenShift cluster where an instance of the application will be scheduled.

        Application will, by default, be visible internally to the OpenShift cluster, and usually only to other applications within the same project. Use `Create route` to make the app public. 

1. Build and deploy from source code contained in a Git repository using a [Source-to-Image](https://github.com/openshift/source-to-image) toolkit.

    ![2](images/s2i-workflow.png)

    See [this video to get s2i presentation](https://www.youtube.com/watch?v=flI6zx9wH6M) and [this section](#s2i) goes to a simple Flask app deployment using s2i. 

1. Build and deploy from source code contained in a Git repository from a Dockerfile.

1. Using Helm charts and helm CLI: Helm can be used as well to define the config files and deploy. Here is a new CI/CD example done from scratch based on the [Reefer ML project simulator code](https://ibm-cloud-architecture.github.io/refarch-reefer-ml).

    *See [getting started](https://docs.bitnami.com/kubernetes/how-to/create-your-first-helm-chart/) with helm guide.*

    * Create helm chart using the command `helm create`

    ```shell
    cd simulator/chart
    helm create kcontainer-reefer-simulator
    ```

    * Change the values.yaml to reflect environment and app settings. Remove Ingress as we will define Openshift route for the app to be visible.

    * In the templates folder modify the deployment.yaml to add env variables section:

    ```yaml
    env:
          - name: PORT
            value: "{{ .Values.service.servicePort }}"
          - name: APPLICATION_NAME
            value: "{{ .Release.Name }}"
          - name: KAFKA_BROKERS
            valueFrom:
              configMapKeyRef:
                name: "{{ .Values.kafka.brokersConfigMap }}"
                key: brokers
          - name: TELEMETRY_TOPIC
            value: "{{ .Values.kafka.telemetryTopicName }}"
          - name: CONTAINER_TOPIC
            value: "{{ .Values.kafka.containerTopicName }}"
          {{- if .Values.eventstreams.enabled }}
          - name: KAFKA_APIKEY
            valueFrom:
              secretKeyRef:
                name: "{{ .Values.eventstreams.apikeyConfigMap }}"
                key: binding
          {{- end }}
    ```

    * Create helm template file for deployment:

    ```shell
    helm template --output-dir templates --namespace eda-demo chart/kcontainer-reefer-simulator/
    ```

    * Push the service.yaml and deployment.yml template to the gitops [repository](https://github.com/ibm-cloud-architecture/refarch-kc-gitops) under the branch `eda-demo/gse-eda-demos.us-east.containers.appdomain.cloud`.
    * In the github repository define secrets environment variables for docker username and password, from your docker hub account.
    * When pushing the repository the gitAction will perform the build.


## Deploy helm / tiller

The goal is to install Tiller server on its own project, and grant it permissions to one or more other projects where Helm Charts will be installed.

See the instructions in [this blog](https://www.openshift.com/blog/getting-started-helm-openshift).

Here is a quick summary of the commands performed

```
oc new-project tiller
oc project tiller
export TILLER_NAMESPACE=tiller
oc process -f https://github.com/openshift/origin/raw/master/examples/helm/tiller-template.yaml -p TILLER_NAMESPACE="${TILLER_NAMESPACE}" -p HELM_VERSION=v2.16.4 | oc create -f -
```

Once deployed and Tiller server running, create a new project and grant tiller edit role to access this new project, and then use helm CLI to deploy the app:

```shell
oc new-project myapp
oc policy add-role-to-user edit "system:serviceaccount:${TILLER_NAMESPACE}:tiller"
```

## Deploy zipkin from docker image

Install it, and expose it with a service

```
oc new-app --docker-image=openzipkin/zipkin

oc expose svc/zipkin
```
A new route is created visible with `oc get routes`. Once the hostname is added to a DNS or /etc/hosts. 

See zipkin architecture [article here](https://zipkin.io/pages/architecture.html)

## Deploy DB2

The Community edition DB2 image on [dockerhub](https://hub.docker.com/r/ibmcom/db2). It includes a predefined DB.

Clone the [DB2 repository](https://github.com/IBM/Db2) to get the helm chart. See [readme](https://github.com/IBM/Db2/tree/develop/deployment) in this repo, for installation using helm but [tiller](#deploy-helm-tiller) needs to be installed before. The repository includes a script: `db2u-install`

Create your own docker image with a shell to create the schema:

```dockerfile
FROM ibmcom/db2

RUN mkdir /var/custom
COPY createschema.sh /var/custom
RUN chmod a+x /var/custom/createschema.sh
```

Deinstalling configuration

```
oc delete -n jbsandbox sa/db2u role/db2u-role rolebinding/db2u-rolebinding
```

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
