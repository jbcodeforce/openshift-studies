# Knative

[Knative](https://knative.dev/) is Kubernetes based platform to develop serverless. Major value proposition is a simplified deployment syntax with automated scale-to-zero and scale-out based on HTTP load. Other interesting characteristics:

* adjust the traffic distribution amongst the service revisions, to support blue/green deployment and canary release
* able to scale to zero: After a defined time of idleness (the so called stable-window) a revision is considered inactive. Now, all routes pointing to the now inactive revision will be pointed to the so-called activator. By default the scale-to-zero-grace-period is 30s, and the stable-window is 60s.
* auto-scaling: By default Knative Serving allows 100 concurrent requests into a pod. This is defined by the container-concurrency-target-default setting in the configmap config-autoscaler in the knative-serving namespace.

Knative consists of the following components:

* Eventing - Management and delivery of events
* Serving - Request-driven compute that can scale to zero

[Redhat knative cookbook](https://redhat-developer-demos.github.io/knative-tutorial/knative-tutorial/index.html).

## Getting started

* Install knative CLI with [brew](https://github.com/knative/homebrew-client).
* Knative CLI run in docker, [see doc here](https://knative.dev/docs/install/install-kn/#kn-container-images)
* To prepare OpenShift [see these instructions](https://docs.openshift.com/container-platform/4.3/serverless/installing_serverless/installing-openshift-serverless.html). 
    * The minimum requirement to use OpenShift Serverless is a cluster with 10 CPUs and 40GB memory. Use operator hub to install Knative serving and eventing. 
    * OpenShift Serverless Operator eventually shows up and its Status ultimately resolves to InstallSucceeded in the openshift-operators namespace.
    * Creating the knative-serving namespace: `oc create namespace knative-serving`, and then within the project create an instance. 
    * Verify the conditions: `oc get knativeserving.operator.knative.dev/knative-serving -n knative-serving --template='{{range .status.conditions}}{{printf "%s=%s\n" .type .status}}{{end}}'`
    * Can do the same of knative-eventing: create a namespace and then an instance using the serverless operator. 
    * Verify with: `oc get knativeeventing.operator.knative.dev/knative-eventing -n knative-eventing --template='{{range .status.conditions}}{{printf "%s=%s\n" .type .status}}{{end}}'`

### Define a service for a given image

oc apply on the following definition:

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: greeter
spec:
 template:
    metadata:
      name: greeter-v2
    spec:
      containers:
      - image: quay.io/rhdevelopers/knative-tutorial-greeter:quarkus
        livenessProbe:
          httpGet:
            path: /healthz
        readinessProbe:
          httpGet:
            path: /healthz
```

Get the detail of the configuration: `oc get configurations.serving.knative.dev greeter`

### Some CLI commands

```shell
# get revisions for a service
oc get rev --selector=serving.knative.dev/service=greeter --sort-by="{.metadata.creationTimestamp}"
oc delete services.serving.knative.dev greeter
```

```shell
# create a service from image
kn service create greeter --image quay.io/rhdevelopers/knative-tutorial-greeter:quarkus
# create a revision
kn service update greeter --env "MESSAGE_PREFIX=Namaste"
kn service describe greeter
kn service delete greeter
# get revisions
kn revision list
kn revision describe greeter-xhwyt-1 

# route
kn route list
```

## Blue/green deployment

Knative offers a simple way of switching 100% of the traffic from one Knative service revision (blue) to another newly rolled out revision (green).
By default new revision receives 100% of the new traffic. To rollback we need to create a yaml with the template.metadata.name to the expected revision and add the following spec.traffic

```yaml
 traffic:
    - tag: v1
      revisionName: greeter-v1
      percent: 100
    - tag: v2
      revisionName: greeter-v2
      percent: 0
    - tag: latest
      latestRevision: true
      percent: 0
```

## Canary release

Knative allows you to split the traffic between revisions

```yaml
 traffic:
    - tag: v1
      revisionName: greeter-v1
      percent: 80
    - tag: v2
      revisionName: greeter-v2
      percent: 20
    - tag: latest
      latestRevision: true
      percent: 0
```

## Knative eventing

There are three primary usage patterns with Knative Eventing:

1. **Source to Sink**: It provides single Sink — that is, event receiving service --, with no queuing, backpressure, and filtering. The Source to Service does not support replies, which means the response from the Sink service is ignored
1. **Channel and subscription**: the Knative Eventing system defines a Channel, which can connect to various backends such as In-Memory, Kafka and GCP PubSub for sourcing the events. Each Channel can have one or more subscribers in the form of Sink services  
  ![1](https://redhat-developer-demos.github.io/knative-tutorial/knative-tutorial-eventing/_images/channels-subs.png)
1. **Broker and Trigger**:  supports filtering of events so subscribers specify interest on certain set of messages that flows into the Broker. For each Broker, Knative Eventing will implicitly create a Knative Eventing Channel. The Trigger gets itself subscribed to the Broker and applies the filter on the messages on its subscribed broker. The filters are applied on the on the Cloud Event attributes of the messages, before delivering it to the interested Sink Services(subscribers).

  ![2](https://redhat-developer-demos.github.io/knative-tutorial/knative-tutorial-eventing/_images/brokers-triggers.png)

To make a project using knative eventing label the namespace with: `kubectl label namespace jbsandbox knative-eventing-injection=enabled`. This will start the filter and ingress pods. 

### Broker and Trigger

See [this tutorial](https://redhat-developer-demos.github.io/knative-tutorial/knative-tutorial-eventing/eventing-trigger-broker.html) for basics trigger and services. This tutorial defines filtering on CloudEvent type named greeting so the two different services can subscribe to.  

When eventing is set the filter and ingress pods are started. To get the address of the broker url: `oc get broker default -o jsonpath='{.status.address.url}'`

Then the approach is to create different sinks, define triggers for each sink on what kind of event attribute to subscribe too, so filtering can occur. The sink will respond depending of the cloud event 'type' attribute for example.
