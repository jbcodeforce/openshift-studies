# Kubernetes FAQ

### what do you understand by kubernetes?
k8s is open source container management / orchestration tool to deploy scaling and descaling of contrainers.
Writing in GO and it is part of the cloud foundation.

The key features:
* automated scheduling to launch container on cluster nodes
* self healing: identify container dying and reschedule them
* automated rollouts and rollbacks
* horizontal scaling and load balancing

---

### Difference between deploying an app on host versus containers?
With host you share OS, libraries, and apps co-exist.

Packaging with container includes OS, config, the libraries and app binary. Containers are made of:
* namespaces: process PID, network, UTS host and domain name isolation, Inter Process Communication isolation, user (container UID are isolated from OS ones), mount tables,
* cgroups: Control group  is a Linux kernel feature that limits, accounts for, and isolates the resource usage (CPU, memory, disk I/O, network, etc.) of a collection of processes.
* layered filesystem: some base layers (e.g linux base layers) can be shared by multiple containers. Layers are usually read-only, and containers make copy if they have to modify the layer
* content: app, libraries...

So containers are more portable.

---

### How k8s simplify container deployment?
By externalizing configuration and requirements, and defining relation between container via configurations: deploymnet, service, secrets, config map...
Developers focus on desired state of the application and defined configuration for k8s to manage.

---

### What is a Heapster?
[Heapster]() is a cluster-wide aggregator of monitoring and event data. It supports Kubernetes natively and works on all Kubernetes setups. It is deprecated. Consider using metrics-server and a third party metrics pipeline to gather Prometheus-format metrics instead.

---

### Which process runs on Kubernetes master node?

`kubectl get pods --all-namespaces --field-selector=spec.nodeName=172.16.40.132`

* Kube-apiserver
* etcd when not clustered
* prometheus node exporter
* logging ELK
* Heapster
* Calico node
* Auth API keys
* Vulnerability advisor

---

### How to do multiple deployment a day?
Rolling updates allow Deployments' update to take place with zero downtime by incrementally updating Pods instances with new ones. By default, the maximum number of Pods that can be unavailable during the update and the maximum number of new Pods that can be created, is one. Updates are versioned and any Deployment update can be reverted to previous.
The approach is to deploy the image to a docker registry and then use:
`kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2`
then we can see the new pods scheduled: `kubectl get pods`.
To rollback to a previous use `kubectl rollout undo deployment/deployment-name`

Using helm we can do `helm rollback <release-name>`

---

### What is the best practices for pod to pod communications?
Don’t specify a hostPort for a Pod unless it is absolutely necessary. When you bind a Pod to a hostPort, it limits the number of places the Pod can be scheduled, because each <hostIP, hostPort, protocol> combination must be unique.
Services are assigned a DNS A record for a name of the form **my-svc.my-namespace.svc.cluster.local**. This resolves to the cluster IP of the Service. So prefer to use DNS name to support this communication.

---

### How to isolate pod to pod communication?
By default, pods are non-isolated; they accept traffic from any source.
Use *network policy* resource to group pods allowed to communicate with each other and other network endpoints. You need a network solution that support network policies. Like Calico.

A simple example of [network policy with nginx](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/).

---

### How to secure internal traffic ?
The first answer is to use to **istio** to simplify the management of certificates and pod to pod communication.
A more manual is to add a reverse-proxy like nginx in each pod. The SSL proxy will terminate HTTPS connection, using the same nginx config. App in container listen to localhost.  

---
### Security practices to Consider
* apply security to environments like host OS of the different nodes
* restrict access to etcd
* implement network segmentation:
* define network policy
* use continuous security Vulnerability scanning
* limit access to nodes
* define resource quota to control overusage
* use private repository and verified images

---

### How RBAC works?
Role-based access control (RBAC) is a method of regulating access to computer or network resources based on the roles of individual users.
A role contains rules that represent a set of permissions. Two types: Role at the namespace level, or ClusterRole. There is no deny rules.

A role binding grants the permissions defined in a role to a user or set of users. It holds a list of subjects (users, groups, or service accounts), and a reference to the role being granted

To assess if RBAC is enabled: `kubectl get clusterroles | grep -i rbac`
[RBAC documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

---

### How to do A/B testing ?
A/B testing deployments consists of routing a subset of users to a new functionality under specific conditions, like HTTP headers, cookie, weight... You need [Istio](http://istio.io) to support A/B testing.

---

### How to support multiple version of the same app?
See A/B testing for one approach and use one of the following deployment strategies:

* **Blue/green** release a new version alongside the old version then switch traffic at the load balancer level (example two different hostnames in the Ingress).


* **Canary** is used when releasing a new version to a subset of users, then proceed to a full rollout. It can be done using two Deployments with common pod labels. But the use of service mesh like istio will define greater control over traffic.

* A **ramped** deployment updates pods in a rolling update fashion, a secondary ReplicaSet is created with the new version of the application, then the number of replicas of the old version is decreased and the new version is increased until the correct number of replicas is reached.

[kubernetes-deployment-strategies examples](https://container-solutions.com/kubernetes-deployment-strategies/)

---

### What are the best practices around Logging?
Logs should have a separate storage and lifecycle independent of nodes, pods, or containers. This concept is called cluster-level-logging. To get container logs use one of:
```
kubectl logs <podname>
kubectl logs <podname> --previous
```
By default, if a container restarts, the kubelet keeps one terminated container with its logs. Everything a containerized application writes to stdout and stderr is handled and redirected somewhere by a container engine to a logging driver.

On machines with systemd, the kubelet and container runtime write to journald. f systemd is not present, they write to .log files in the /var/log directory.

For cluster-level logging three choices:
* Use a node-level logging agent that runs on every node: another container which access logs at node level and exposes logs or pushes logs to a backend. Better define a DaemonSet for that. This is the most common and encouraged approach for a Kubernetes cluster. For GKE [Stackdriver Logging](https://kubernetes.io/docs/tasks/debug-application-cluster/logging-stackdriver/), or [elasticsearch and kibana](https://kubernetes.io/docs/tasks/debug-application-cluster/logging-elasticsearch-kibana/)
* Include a dedicated sidecar container for logging in an application pod. The sidecar streams application logs to its own stdout. It runs a logging agent, which is configured to pick up logs from an application container. Each sidecar container could tail a particular log file from a shared volume and then redirect the logs to its own stdout stream
* Push logs directly to a backend from within an application.

If you have an application that writes to a single file, it’s generally better to set /dev/stdout as destination rather than implementing the streaming sidecar container approach.

---

### How is persistent volume allocated?
Before you can deploy most Kubernetes-based application, you must first create system-level storage from the underlying infrastructure.
**hostPath** Persistence Volume uses local to the worker node filesystem. So create a host directory on all worker nodes in your cluster. `hostPath` PV could not failover to other worker node.
**NFS** is shared between worker nodes. See this [article to set up NFS on ubuntu](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-16-04). When crating the PV, use the key value pairs:
```
server - 172.29.2.8
path - /data/local/share
```
Then define PVC and configure your application to access the PVC defined.

---

### How to debug 503?
See the troubleshooting [here](https://github.com/ibm-cloud-architecture/refarch-integration/blob/master/docs/icp/troubleshooting.md#accessing-an-app-expose-with-ingress-503-service-temporarily-unavailable)

---

### how to see ingress controller is up and running in k8s?
* Get the list of ingress: `kubectl get ing -n greencompute`
* Get a specific ingress: `kubectl describe ing -n greencompute gc-customer-ms-greencompute-customerms`
 ```
 Name:             gc-customer-ms-greencompute-customerms
Namespace:        greencompute
Address:          172.16.40.131
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host                 Path  Backends
  ----                 ----  --------
  customer.green.case  
                       /   gc-customer-ms-greencompute-customerms:9080 (<none>)
Annotations:
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  42m   nginx-ingress-controller  Ingress greencompute/gc-customer-ms-greencompute-customerms
 ```
* Check the Ingress Controller Logs: `kubectl get pods -n kube-system` then `kubectl logs nginx-ingress-controller-gvcj5 -n kube-system`. Search for message with "Event ... Kind: Ingress..."
* Verify the nginx configuration with a command like: ` kubectl exec -it -n kube-system nginx-ingress-controller-gvcj5  cat /etc/nginx/nginx.conf`


Another [source](https://github.com/kubernetes/ingress-nginx/blob/master/docs/troubleshooting.md) of information

---

### What is [Calico](https://www.projectcalico.org/)?
It uses a IP-only routing connectivity to connect containers. It does not need tunneling and virtual multicast or broadcast. It is an implementation of the Container Network Interface protocol: a plug and play networking capability with the goal to support adding and removing container easily with dynamic IP address assignment.
Each container has an IP address as Node has one too. When a pod is scheduled the orchestrator calls CNI plugin, Calico, which defines the IP address to use, and persists the states in `etcd`. There is a container, called `felix`, which runs in each host and modifies the linux kernel to reference the new IP address in IP routing table. There is also a BGP component that propagated the IP address to all the node of the cluster so they can send connection to this new IP address. This is to flatten the network.

### [Container Network Interface](https://github.com/containernetworking/cni)?
Apply the network as software approach to container but focusing of connecting workload (or remove it) instead of doing lower level of network configuration.
Network policy for security is not in CNI.

<img src="assets/docs/studies/k8s/cni-k8s.png"></img>

CNI supports two commands: ADD and DEL that orchestrator can use to specify a container is added or removed. On the ADD command, CNI create a network interface (visible via `ifconfig`) within the network namespace and add new route (`route -n`) in the host so traffic can be routed to the container and in the container define default route to be able to get out.

<img src="assets/docs/studies/k8s/cni-pod-net.png"></img>

---

### What is the number of master needed to support HA?
It is linked to the cluster size, and the expected fault tolerance which is computed as (#_of_master - 1) / 2. It should be at least 1. You must have an odd number of masters in your cluster. Majority is the number of master nodes needed for the cluster to be able to operate. For a cluster of 6 worker nodes, you need a majority of 4 and then a fault tolerance of 2.

---

### How to use k8s in docker edge?

[This tutorial](https://rominirani.com/tutorial-getting-started-with-kubernetes-with-docker-on-mac-7f58467203fd) to get Edge and k8s started. The following commands can help once docker + k8s are running too:
```
# Set the cluster - Use tls-verify=true to avoid: Unable to connect to the server: x509: certificate signed by unknown authority

$ kubectl config set-cluster docker-for-desktop --server=https://kubernetes:6443 --insecure-skip-tls-verify=true

# Error from server (Forbidden): services is forbidden: User "system:anonymous" cannot list services in the namespace "kube-system"
```

 

