# Kubeless

[Kubeless](http://kubeless.io/) is a Kubernetes-native serverless framework. It runs on top of your Kubernetes cluster and allows you to deploy small unit of code without having to build container images. With kubeless you can build advanced applications that tie together services using functions.

## TL;DR;

```bash
$ helm install tc/kubeless --namespace kubeless
```

## Introduction

This chart bootstraps a [Kubeless](https://github.com/kubeless/kubeless) and a [Kubeless-UI](https://github.com/kubeless/kubeless-ui) deployment on a [Kubernetes](http://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

## Prerequisites

- Kubernetes 1.7+ with Beta APIs enabled
- PV provisioner support in the underlying infrastructure

## Installing the Chart

To install the chart with the release name `kubeless`:

```bash
$ helm install --name kubeless tc/kubeless --namespace kubeless
```

The command deploys Kubernetes on the Kubernetes cluster in the default configuration. The [configuration](#configuration) section lists the parameters that can be configured during installation.

> **Tip**: List all releases using `helm list`

## Uninstalling the Chart

To uninstall/delete the `kubeless` deployment:

```bash
$ helm delete kubeless
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

The following tables lists the configurable parameters of the Kubeless chart and their default values.

|         Parameter                       |             Description             |                         Default                          |
|-----------------------------------------|-------------------------------------|----------------------------------------------------------|
| `controller.deployment.image.repository`| Controller image                    | `bitnami/kubeless-controller@sha256`                     |
| `controller.deployment.image.pullPolicy`| Controller image pull policy        | `IfNotPresent`                                           |
| `controller.deployment.replicaCount`    | Number of replicas                  | `1`                                                      |
| `ui.deployment.ui.image.repository`     | Kubeless UI image                   | `bitnami/kubeless-ui`                                    |
| `ui.deployment.ui.image.pullPolicy`     | Kubeless UI image pull policy       | `IfNotPresent`                                           |
| `ui.deployment.proxy.image.repository`  | Proxy image                         | `kelseyhightower/kubectl`                                |
| `ui.deployment.proxy.image.pullPolicy`  | Proxy image pull policy             | `IfNotPresent`                                           |
| `ui.deployment.replicaCount`            | Number of replicas                  | `1`                                                      |
| `ui.service.name`                       | Service name                        | `ui-port`                                                |
| `ui.service.type`                       | Kubernetes service name             | `NodePort`                                               |
| `ui.service.externalPort`               | Service external port               | `3000`                                                   |
| `zookeeper.statefulSet.image.repository`| Zookeeper image                     |  `bitnami/zookeeper@sha256`                              |
| `zookeeper.statefulSet.image.pullPolicy`| Zookeeper image pull policy         | `IfNotPresent`                                           |
| `zookeeper.statefulSet.replicaCount`    | Number of replicas                  | `1`                                                      |
| `kafka.statefulSet.image.repository`    | Kafka image                         |  `bitnami/kafka@sha256`                                  |
| `kafka.statefulSet.image.pullPolicy`    | Kafka image pull policy             | `IfNotPresent`                                           |
| `kafka.statefulSet.replicaCount`        | Number of replicas                  | `1`                                                      |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```bash
$ helm install --name kubeless --set service.name=ui-service,service,externalPort=4000 --namespace kubeless tc/kubeless
```

The above command sets the Kubeless service name to `ui-service` and the external port to `4000`.

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```bash
$ helm install --name kubeless -f values.yaml --namespace kubeless tc/kubeless
```

> **Tip**: You can use the default [values.yaml](values.yaml)

## Persistence

The Kafka and Zookeeper images stores the data and configurations at the `/bitnami/kafka` and `/bitnami/zookeeper` path of the container.
