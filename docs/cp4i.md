# Some notes on Cloud Pak for integration

* [Marketing page](https://www.ibm.com/cloud/cloud-pak-for-integration)
* [Installation instructions from product doc](https://www.ibm.com/support/knowledgecenter/en/SSGT7J)

Real integration project will use multiple instances of MQ, event streams, API management. Cloud Pak helps to manage n instances.

## Common problems CP4I addresses

* Keep integration while moving to the cloud
* easily connect applications and data across multiple clouds 
* manage coherence while  LOBs and IT teams are independently proliferating their own integration model, capabilities and platform
* How to move backend capabilities are now being provided by SaaS provider
* Scale new volume of demand

## Breaking up the ESB

SOA was typically implemented using the ESB pattern which provides standardized synchronous connectivity to back-end systems typically over web services. Some challenges surface:

* ESB patterns often formed a single infrastructure for the whole enterprise, with tens or hundreds of integrations installed on a production server cluster.
* Few interfaces could be reused from one project to another, yet the creation and maintenance of interfaces was prohibitively expensive for the scope of any one project to take on.
* Cross-enterprise initiatives like SOA and its underlying ESB were hard to fund, and often that funding only applied to services that would be reusable enough to cover the cost of creation.

[The agile integration architecture](https://www.ibm.com/cloud/integration/agile-integration/) supports using a lightweight integration runtimes to implement a container based integration.
The drive for changes can be summarized by the need for fine grained deployment, the adoption of cloud native infrastructure, and decentralized ownership of the integration.

Container brings the operational consistency for any type of product and component.


## Common services

* LDAP integration
* UI to manage all the components
* Consistent monitoring dashboard
* Seamless integration between APP Connect flow and API mgt.
* MQ connectivity in App Connect
* Common loggging, troubleshooting, and security

The asset repository will keep a catalog of integration, APIs,... These assets do not have to be physically stored in the Asset Repository. The physical storage could be GitHub. These asset are treated as "Read-Only".

## Installation

See [Knowledge Center](https://www.ibm.com/support/knowledgecenter/en/SSGT7J_20.4/install/install.html).

All is done via operators now:

```
oc get operators

# ibm-apiconnect.openshift-operators                              5d20h
# ibm-appconnect.openshift-operators                              5d20h
# ibm-cert-manager-operator.ibm-common-services                   5d20h
# ibm-common-service-operator.openshift-operators                 5d20h
# ibm-commonui-operator-app.ibm-common-services                   5d19h
# ibm-eventstreams.openshift-operators                            5d19h
# ibm-iam-operator.ibm-common-services                            5d19h
# ibm-ingress-nginx-operator-app.ibm-common-services              5d19h
# ibm-integration-asset-repository.openshift-operators            5d20h
# ibm-integration-operations-dashboard.openshift-operators        5d20h
# ibm-integration-platform-navigator.openshift-operators          5d20h
# ibm-licensing-operator-app.ibm-common-services                  5d19h
# ibm-management-ingress-operator-app.ibm-common-services         5d19h
# ibm-metering-operator-app.ibm-common-services                   5d10h
# ibm-mongodb-operator-app.ibm-common-services                    5d19h
# ibm-monitoring-exporters-operator-app.ibm-common-services       5d19h
# ibm-monitoring-grafana-operator-app.ibm-common-services         5d19h
# ibm-monitoring-prometheusext-operator-app.ibm-common-services   5d19h
# ibm-mq.openshift-operators                                      5d20h
# ibm-namespace-scope-operator.ibm-common-services                5d20h
# ibm-odlm.ibm-common-services                                    5d20h
# ibm-platform-api-operator-app.ibm-common-services               5d19h

```

## MQ

* [MQ Marketing page](https://www.ibm.com/cloud/cloud-pak-for-integration/enterprise-messaging)
* [MQ on cloud product tour tutorial](https://www.ibm.com/cloud/garage/dte/producttour/ibm-mq-ibm-cloud-product-tour)
* [Personal MQ summary](https://ibm-cloud-architecture.github.io/refarch-eda/technology/mq/)

My repos:

* [Container inventory legacy app](https://github.com/ibm-cloud-architecture/refarch-container-inventory)
* [MQ Messaging Solution](https://github.com/ibm-cloud-architecture/refarch-mq-messaging)

Interesting reusable code



## App Connect


## API management

## Aspera

