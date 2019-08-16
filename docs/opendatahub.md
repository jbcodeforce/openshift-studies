# Open Data Hub Studies

[Open Data Hub main site](https://opendatahub.io)

## Summary

The goal of Open Data Hub is to provide open source AI tools for running large and distributed AI workloads on Openshift Container platform. While AI Library is to provide ML models as a service on Openshift.

In general, an AI workflow includes most of the steps shown in figure below:

![](odh-figure-1.png)

For data storage and availability, ODH provides Ceph, with multi protocol support including block, file and S3 object API support, both for persistent storage within the containers and as a scalable object storage data lake that AI applications can store and access data from.

!!! notes
        [Ceph](https://docs.ceph.com/docs/master/start/intro/) delivers a self-managed, self-scaling, and self-healing storage infrastructure using storage cluster. A key Ceph architectural tenet is to have no single point of failure (SPoF) in the system.

## Installation on Openshift

The latest version of the Open Data Hub operator project is located here: https://gitlab.com/opendatahub/opendatahub-operator

Install Ceph with the [rook operator](https://www.redhat.com/en/blog/rook-ceph-storage-operator-now-operatorhubio) using [these instructions](https://opendatahub.io/arch.html#ceph-installation-with-the-rook-operator).

The following github has the configurations: [https://github.com/rook](https://github.com/rook)

To validate pods 

```
oc get pods -n rook-ceph-system
oc get pods -n rook-ceph
```

Some errors:
