# Knative

[Knative](https://knative.dev/) is Kubernetes based platform to develop serverless. Major value proposition is a simplified deployment syntax with automated scale-to-zero and scale-out based on HTTP load. Other interesting characteristics:

* **adjust the traffic distribution** amongst the service revisions, to support blue/green deployment and canary release
* able to **scale to zero**: After a defined time of idleness (the so called stable-window) a revision is considered inactive. Now, all routes pointing to the now inactive revision will be pointed to the so-called activator. By default the scale-to-zero-grace-period is 30s, and the stable-window is 60s.
* **auto-scaling**: By default Knative Serving allows 100 concurrent requests into a pod. This is defined by the container-concurrency-target-default setting in the configmap config-autoscaler in the knative-serving namespace.

Knative consists of the following components:

* Eventing - Management and delivery of events
* Serving - Request-driven compute that can scale to zero

## Knative serving

Knative Serving controller creates a Configuration, a Revision, and a Route. The Knative Configuration maintains the desired state of your deployment, providing a clean separation of code and configuration. Every change to the application configuration creates a new Knative Revision.

## Value propositions

Like any serverless (AWS lambda, openwhisk..) the benefits are:

* instantly spin up enough server resources to handle tens of thousands of incoming requests
* more expensive on compute time, but cost zero after
* spin up a completely separate version of the site to do some prototyping, no need to worry about forgetting a test running forever
*  No Kubernetes configurations, no load balancers, no auto scaling rules

### Challenges (aws lambda that may apply to knative)

* web socket support, SMTP session
* With AWS no control of the OS, so difficult to bring your libraries. Knative as container fixes this.
* AWS: functions with less TAM have slower CPU speed. This can swing a lot between 5ms to 500ms response time. Charged by 100ms increment
* Using SPA app like React or Angular, the page will be downloaded from a Content CDN server, which leads to have a delay between this web server and the function. If you want <50ms response times, then hosting your backend behind API Gateway is not for function.
* Need non serverless services like VPC, NAT gateways to access other service like storage. 
* to get a granular record of what Lambdas are running, how many times and for how long, measurement and Logging products are not serverless, so cost a lot. Specially as you may not control the information sent to the logger (Cloudwatch).
* Idempotence: AWS Lambda, can execute the same requests more than once. Which will generate multiple records written into the DB. This is due to retries on errors, and event source set at least once delivery.
Use request identifiers to implement idempotent Lambda functions that do not break if a function invocation needs to be retried. Make sure to pass the original request id to all subsequent calls. The original request id is generated as early as possible in the chain. If possible on the client side. Avoid generating your own ids.
* Time limit on execution to prevent deadlocked function. Difficult to see those functions as they disappear.
* COLD start: if your function hasn’t been run in a while, a pending request needs to wait for it to be initialized before it can be served: 3 to 5 seconds. nothing the end-user touches could be hosted on Lambda after all
* view Lambda mainly in terms of what it can do as a high-end parallel architecture 

## Getting started

* Install Knative CLI with [brew](https://github.com/knative/homebrew-client). It will be accessible via `kn`
* Or use Knative CLI running in docker, [see doc here](https://knative.dev/docs/install/install-kn/#kn-container-images)
* To prepare OpenShift [see the serverless install instructions](https://docs.openshift.com/container-platform/4.3/serverless/installing_serverless/installing-openshift-serverless.html). 

    * The minimum requirement to use OpenShift Serverless is a cluster with 10 CPUs and 40GB memory. Use operator hub to install Knative serving and eventing servers and brokers. 
    * OpenShift Serverless Operator eventually shows up and its Status ultimately resolves to InstallSucceeded in the openshift-operators namespace.
    * Creating the knative-serving namespace: `oc create namespace knative-serving`, and then within the project itself, create an instance. 
    * Verify the conditions: `oc get knativeserving.operator.knative.dev/knative-serving -n knative-serving --template='{{range .status.conditions}}{{printf "%s=%s\n" .type .status}}{{end}}'`
    You should get:

    ```
    DependenciesInstalled=True
    DeploymentsAvailable=True
    InstallSucceeded=True
    Ready=True
    ```
    * With a yaml file: and `oc apply` it.
    
    ```yaml
    apiVersion: operator.knative.dev/v1alpha1
    kind: KnativeServing
    metadata:
      name: knative-serving
      namespace: knative-serving
      spec:
        high-availability:
          replicas: 2
    ```
    
    *We can do the same for knative-eventing: create a namespace and then an instance using the serverless operator. 
    * Verify with: `oc get knativeeventing.operator.knative.dev/knative-eventing -n knative-eventing --template='{{range .status.conditions}}{{printf "%s=%s\n" .type .status}}{{end}}'`

### Define a service for a given image

oc apply the following definition:

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

### Knative and Quarkus app

Be sure to have the kubernetes or openshit plugin: `./mvnw quarkus:add-extension -Dextensions="openshift"`
Add the following property:

```properties
quarkus.kubernetes.deployment-target=knative
```

When doing `mvn package` a knative.yaml file is created under target/kubernetes

Other example of creating deployment:

```
mvn -Dcontainer.registry.url='https://index.docker.io/v1/' \
> -Dcontainer.registry.user='jbcodefore' \
> -Dcontainer.registry.password='XXXXXXXYYYYYYYZZZZZZZZ' \
> -Dgit.source.revision='master' \
> -Dgit.source.repo.url='https://github.com/quarkusio/quarkus-quickstarts.git' \
> -Dapp.container.image='quay.io/jbcodefore/quarkus-greetings' package
```

This command creates the resource files in `target/kubernetes` directory

Deploy the service `oc apply --recursive --filename target/kubernetes/`

Some RedHat [article](https://developers.redhat.com/blog/2019/04/09/from-zero-to-quarkus-and-knative-the-easy-way/).

## Knative Eventing


## Some CLI commands

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

1. **Source to Sink**: It provides single Sink — that is, event receiving service --, with no queuing, back-pressure, and filtering. The Source to Service does not support replies, which means the response from the Sink service is ignored
1. **Channel and subscription**: the Knative Eventing system defines a Channel, which can connect to various backends such as In-Memory, Kafka and GCP PubSub for sourcing the events. Each Channel can have one or more subscribers in the form of Sink services  
  ![1](https://redhat-developer-demos.github.io/knative-tutorial/knative-tutorial-eventing/_images/channels-subs.png)
1. **Broker and Trigger**:  supports filtering of events so subscribers specify interest on certain set of messages that flows into the Broker. For each Broker, Knative Eventing will implicitly create a Knative Eventing Channel. The Trigger gets itself subscribed to the Broker and applies the filter on the messages on its subscribed broker. The filters are applied on the on the Cloud Event attributes of the messages, before delivering it to the interested Sink Services(subscribers).

  ![2](https://redhat-developer-demos.github.io/knative-tutorial/knative-tutorial-eventing/_images/brokers-triggers.png)

To make a project using knative eventing label the namespace with: `kubectl label namespace jbsandbox knative-eventing-injection=enabled`. This will start the filter and ingress pods.

### Broker and Trigger

See [this tutorial](https://redhat-developer-demos.github.io/knative-tutorial/knative-tutorial-eventing/eventing-trigger-broker.html) for basics trigger and services. This tutorial defines filtering on CloudEvent type named greeting so the two different services can subscribe to.  

When eventing is set the filter and ingress pods are started. To get the address of the broker url: `oc get broker default -o jsonpath='{.status.address.url}'`

Then the approach is to create different sinks, define triggers for each sink on what kind of event attribute to subscribe too, so filtering can occur. The sink will respond depending of the cloud event 'type' attribute for example.

## Troubleshooting

Source of knowledge [Debugging issues with your application](https://knative.dev/docs/serving/debugging-application-issues/).

* Revision failed

```
oc get configurations.serving.knative.dev item-kafka-producer
NAME                  LATESTCREATED               LATESTREADY   READY     REASON
item-kafka-producer   item-kafka-producer-65kbv                 False     RevisionFailed
```

## Sources

* [Redhat knative tutorial](https://redhat-developer-demos.github.io/knative-tutorial/knative-tutorial/index.html).
* [Knative Cookbook](https://learning.oreilly.com/library/view/knative-cookbook)