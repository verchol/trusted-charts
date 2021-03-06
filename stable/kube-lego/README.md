# kube-lego

**It is based on Helm community chart [kube-lego](https://github.com/kubernetes/charts/tree/master/stable/kube-lego)**

[kube-lego](https://github.com/jetstack/kube-lego) automatically requests certificates for Kubernetes Ingress resources from Let's Encrypt.

## TL;DR;

```console
$ helm install tc/kube-lego
```

## Introduction

This chart bootstraps a kube-lego deployment on a [Kubernetes](http://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

## Prerequisites
  - Kubernetes 1.4+ with Beta APIs enabled

## Installing the Chart

To install the chart with the release name `kube-lego`:

```console
$ helm install --name kube-lego tc/kube-lego
```

The command deploys kube-lego on the Kubernetes cluster in the default configuration. The [configuration](#configuration) section lists the parameters that can be configured during installation.

> **Tip**: List all releases using `helm list`

## Uninstalling the Chart

To uninstall/delete the `kube-lego` deployment:

```console
$ helm delete kube-lego
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

The following table lists the configurable parameters of the chart and their default values.

Parameter | Description | Default
--------- | ----------- | -------
`config.LEGO_EMAIL` | email address to use for registration with Let's Encrypt | none
`config.LEGO_URL` | Let's Encrypt API endpoint | `https://acme-staging.api.letsencrypt.org/directory` (staging)
`config.LEGO_PORT` | kube-lego port | `8080`
`image.repository` | kube-lego container image repository | `jetstack/kube-lego`
`image.tag` | kube-lego container image tag | `0.1.3`
`image.pullPolicy` | kube-lego container image pull policy | `IfNotPresent`
`nodeSelector` | node labels for pod assignment | `{}`
`podAnnotations` | annotations to be added to pods | `{}`
`replicaCount` | desired number of pods | `1`
`resources` | kube-lego resource requests and limits (YAML) |`{}`
`rbac.create` | Create a role and serviceaccount | `false`
`rbac.serviceAccountName` | serviceaccount name to use if `rbac.create` is false | `default`

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```console
$ helm install --name kube-lego \
  --set config.LEGO_EMAIL=you@domain.tld \
    tc/kube-lego
```

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```console
$ helm install --name kube-lego -f values.yaml tc/kube-lego
```

> **Tip**: You can use the default [values.yaml](values.yaml)
