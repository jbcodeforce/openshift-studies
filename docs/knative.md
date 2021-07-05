# Knative

[Knative](https://knative.dev/) is Kubernetes based platform to develop serverless. Major value proposition is a simplified 
deployment syntax with automated scale-to-zero and scale-out based on HTTP load. 
Other interesting characteristics are:

* **adjust the traffic distribution** amongst the service revisions, to support blue/green deployment and canary release
* able to **scale to zero**: After a defined time of idleness (the so called stable-window) a revision is considered inactive. Now, all routes pointing to the now inactive revision will be pointed to the so-called activator. By default the scale-to-zero-grace-period is 30s, and the stable-window is 60s.
* **auto-scaling**: By default Knative Serving allows 100 concurrent requests into a pod. This is defined by the 
`container-concurrency-target-default` setting in the configmap `config-autoscaler` in the knative-serving namespace.

Knative consists of the following components:

* [Serving](#knative-serving) - Request-driven compute that can scale to zero
* [Eventing](#knative-eventing) - Management and delivery of events

With OpenShift it is called OpenShift Serverless, and offers operators for functions, serving and eventing. 
Integrated with Apache Camel connectors. 

## Value propositions

Like any serverless (AWS lambda, openwhisk..) the benefits are:

* instantly spin up enough server resources to handle tens of thousands of incoming requests
* more expensive on compute time, but cost zero after
* spin up a completely separate version of the site to do some prototyping, no need to worry about forgetting a test running forever
* no Kubernetes configurations, no load balancers, no auto scaling rules

### Challenges (based on AWS lambda but may apply to knative)

* web socket support, SMTP session
* With AWS no control of the OS, so difficult to bring your libraries. Knative as container fixes this challenge.
* AWS: functions with less RAM have slower CPU speed. This can swing a lot between 5ms to 500ms response time. Charged by 100ms increment
* Using SPA app like React or Angular, the page will be downloaded from a ContentDN server, which leads to have a delay between this web server and the function. 
If you want <50ms response times, then hosting your backend behind API Gateway is not usable for function.
* App that needs non serverless services like VPC, NAT gateways to access other service like storage. 
* to get a granular view of how long Lambdas are running, how many times... Measurement and Logging products are not serverless, so cost a lot. 
Specially as you may not control the information sent to the logger (like CloudWatch).
* Idempotence: AWS Lambda, can execute the same requests more than once. Which will generate multiple records written into the DB. 
This is due to retries on error, and event source set at least once delivery.
Use request identifiers to implement idempotent Lambda functions that do not break if a function invocation needs to be retried. 
Make sure to pass the original request id to all subsequent calls. The original request id is generated as early as possible in the chain. 
If possible on the client side. Avoid generating your own ids.
* Time limit on execution to prevent deadlocked function. Difficult to see those functions as they disappear quickly.
* COLD start: if your function hasn’t been run in a while, a pending request needs to wait for it to be initialized before it can be served: 3 to 5 seconds. 
Nothing the end-user touches could be hosted on Lambda after all
* view Lambda mainly in terms of what it can do as a high-end parallel architecture 

## Getting started

* Install Knative CLI with [brew](https://github.com/knative/homebrew-client). It will be accessible via `kn`
* Or use Knative CLI running in docker, [see doc here](https://knative.dev/docs/install/install-kn/#kn-container-images)
* To prepare OpenShift [see the serverless install instructions](https://docs.openshift.com/container-platform/4.7/serverless/serverless-getting-started.html). 

    * The minimum requirement to use OpenShift Serverless is a cluster with 10 CPUs and 40GB memory. Use operator hub to install Knative serving and eventing servers and brokers. 
    * OpenShift Serverless Operator eventually shows up and its Status ultimately resolves to InstallSucceeded in the openshift-operators namespace.
    * Creating the knative-serving namespace: `oc create namespace knative-serving`, and then within the project itself, create an instance. 
    * Verify the conditions: `oc get knativeserving.operator.knative.dev/knative-serving -n knative-serving --template='{{range .status.conditions}}{{printf "%s=%s\n" .type .status}}{{end}}'`
    You should get:

    ```properties
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
    
    * We can do the same for `knative-eventing`: create a namespace and then an instance using the serverless operator. 
    * Verify with: `oc get knativeeventing.operator.knative.dev/knative-eventing -n knative-eventing --template='{{range .status.conditions}}{{printf "%s=%s\n" .type .status}}{{end}}'`

## Knative serving

Knative Serving controller creates a Configuration, a Revision, and a Route. The Knative Configuration maintains 
the desired state of your deployment, providing a clean separation of code and configuration. 
Every change to the application configuration creates a new Knative Revision.

### Define a service for a docker image

```shell
# tutorial example
kn service create greeter --image quay.io/rhdevelopers/knative-tutorial-greeter:quarkus

kn service list
# Update service with env variable
kn service update greeter  --env "MESSAGE_PREFIX=Namaste"
kn service describe greeter
# Call service
http $(kn service describe greeter -o url)
# Get revision list 
kn revision list 
# Delete a revision
kn revision delete <revision-id>
# List route
kn route list
# Destroy
kn service delete greeter

```

A revision is the immutable application and configuration state that gives you the capability to roll back 
to any last known good state.


### Define a service using yaml

oc apply the following definition. The Knative Service is associated with its apiVersion and kind `service.serving.knative.dev`. 
Liveness and readiness ports are infered from deployment.


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

When doing `mvn package` a `knative.yaml` file is created under `target/kubernetes`

```
mvn -Dcontainer.registry.url='https://index.docker.io/v1/' \
> -Dcontainer.registry.user='jbcodeforce' \
> -Dcontainer.registry.password='XXXXXXXYYYYYYYZZZZZZZZ' \
> -Dgit.source.revision='master' \
> -Dgit.source.repo.url='https://github.com/quarkusio/quarkus-quickstarts.git' \
> -Dapp.container.image='quay.io/jbcodefore/quarkus-greetings' package
```

This command creates the resource files in `target/kubernetes` directory

Deploy the service `oc apply --recursive --filename target/kubernetes/`

Some RedHat [article](https://developers.redhat.com/blog/2019/04/09/from-zero-to-quarkus-and-knative-the-easy-way/).


### Blue/green deployment

Knative offers a simple way of switching 100% of the traffic from one Knative service revision (blue) to another newly rolled out revision (green).
By default new revision receives 100% of the new traffic. To rollback we need to create a yaml with the `template.metadata.name` to the expected 
revision and add the following spec.traffic

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

### Canary release

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

To use Knative eventing, we need to create a `knative-eventing` project and then in the Knative (Red Hat OpenShift serverless) operator create one instance.

```yaml
apiVersion: operator.knative.dev/v1alpha1
kind: KnativeEventing
metadata:
  name: knative-eventing
  namespace: knative-eventing
spec: {}
```

Verify it runs successfully by looking at the predefeind services:

```shell 
kubectl api-resources --api-group='sources.knative.dev'
```

There are three primary usage patterns with Knative Eventing:

1. **Source to Sink**: It provides single Sink — that is, event receiving service --, with no queuing, back-pressure, and filtering. The Source to Service does not support replies, which means the response from the Sink service is ignored
2. **Channel and subscription**: the Knative Eventing system defines a Channel, which can connect to various backends such as In-Memory, Kafka and GCP PubSub for sourcing the events. Each Channel can have one or more subscribers in the form of Sink services  
  
   ![1](https://redhat-developer-demos.github.io/knative-tutorial/knative-tutorial/eventing/_images/channels-subs.png)

   When forwarding event to subscribers the channel transforms the event data as per CloudEvent specification

To make a project using knative eventing label the namespace with: `kubectl label namespace jbsandbox knative-eventing-injection=enabled`. This will start the filter and ingress pods.

### Source and sink

From Red Hat tutorial, the commands to test knative eventing

```shell
# Under current namespace
# 1- Create sink
kn service create eventinghello \
  --concurrency-target=1 \
  --image=quay.io/rhdevelopers/eventinghello:0.0.2
# A pod is created with name like  eventinghello-sdnfp-1-deployment-...

# 2 Create source as a generator of json message
kn source ping create eventinghello-ping-source \
  --schedule "*/2 * * * *" \
  --sink ksvc:eventinghello
# Verify source start to send message
kn source ping list
# Verify sink got messages
oc logs <sink-app-pod>
# clean first the source then the sink
kn source ping delete eventinghello-ping-source
kn service delete eventinghello
```

### Channel and Subscribers

Here is a simple set of steps to demo channel and subscriber on the ping source

```shell
# Create the in memory channel 
kn channel create eventinghello-ch
kn channel list
# Create source 
kn source ping create eventinghello-ping-source \
  --schedule "*/2 * * * *" \
  --sink channel:eventinghello-ch
kn source ping list
# create a subscriber service
kn service create eventinghelloa --concurrency-target=1 --revision-name=eventinghelloa-v1 --image=quay.io/rhdevelopers/eventinghello:0.0.2
# Add the subscription
kn subscription create  eventinghelloa-sub --channel eventinghello-ch   --sink eventinghelloa
kn subscription list
# Clean up
kn service delete eventinghello
kn subscription delete eventinghelloa-sub
kn source ping delete eventinghello-ping-source
```

### Broker and Trigger

See [this tutorial](https://redhat-developer-demos.github.io/knative-tutorial/knative-tutorial/eventing/eventing-trigger-broker.html) for basics trigger and services explanation. 

**Broker and Trigger** supports filtering of events so subscribers specify interest on certain set of messages that flows into the Broker. For each Broker, Knative Eventing will implicitly create a Knative Eventing Channel. The Trigger gets itself subscribed to the Broker and applies the filter on the messages on its subscribed broker. The filters are applied on the on the Cloud Event attributes of the messages, before delivering it to the interested Sink Services(subscribers).

![2](https://redhat-developer-demos.github.io/knative-tutorial/knative-tutorial/eventing/_images/brokers-triggers.png)

When eventing is set the filter and ingress pods are started. To get the address of the broker url:
 `oc get broker default -o jsonpath='{.status.address.url}'`

Then the approach is to create different sinks, define triggers for each sink on what kind of event attribute 
to subscribe to, so filtering can occur. The sink will respond depending of the cloud event 'type' attribute for example.


## Some other CLI commands

```shell
# get revisions for a service
oc get rev --selector=serving.knative.dev/service=greeter --sort-by="{.metadata.creationTimestamp}"
oc delete services.serving.knative.dev greeter
```

```shell
# get revisions
kn revision list
kn revision describe greeter-xhwyt-1 

# route
kn route list
```

## Troubleshooting

Source of knowledge [Debugging issues with your application](https://knative.dev/docs/serving/debugging-application-issues/).

* Revision failed

```
oc get configurations.serving.knative.dev item-kafka-producer
NAME                  LATESTCREATED               LATESTREADY   READY     REASON
item-kafka-producer   item-kafka-producer-65kbv                 False     RevisionFailed
```

## Sources

* [Red Hat knative tutorial on github](https://redhat-developer-demos.github.io/knative-tutorial/knative-tutorial/index.html).
* [Knative Cookbook on O'Reilly portal](https://learning.oreilly.com/library/view/knative-cookbook)
* [REd Hat Knative Tutorial](https://redhat-developer-demos.github.io/knative-tutorial/knative-tutorial/index.html)
* [Quarkus Funqy Knative events binding guide](https://quarkus.io/guides/funqy-knative-events)