# MongoDB Helm Chart

**It is based on Helm community chart [mongodb-replicaset](https://github.com/kubernetes/charts/tree/master/stable/mongodb-replicaset)**

## Prerequisites Details
* Kubernetes 1.6+ with Beta APIs enabled.
* PV support on the underlying infrastructure.

## StatefulSet Details
* https://kubernetes.io/docs/concepts/abstractions/controllers/statefulsets/

## StatefulSet Caveats
* https://kubernetes.io/docs/concepts/abstractions/controllers/statefulsets/#limitations

## Chart Details

This chart implements a dynamically scalable [MongoDB replica set](https://docs.mongodb.com/manual/tutorial/deploy-replica-set/)
using Kubernetes StatefulSets and Init Containers.

## Installing the Chart

To install the chart with the release name `mongodb-rs`:

```console
$ helm install tc/mongodb-replicaset --name mongodb-rs --namespace=mongodb
```

## Configuration

The following tables lists the configurable parameters of the mongodb chart and their default values.

| Parameter                       | Description                                                               | Default                                             |
| ------------------------------- | ------------------------------------------------------------------------- | --------------------------------------------------- |
| `replicaSet`                    | Name of the replica set                                                   | rs0                                                 |
| `replicas`                      | Number of replicas in the replica set                                     | 3                                                   |
| `port`                          | MongoDB port                                                              | 27017                                               |
| `installImage.name`             | Image name for the init container that establishes the replica set        | gcr.io/google_containers/mongodb-install            |
| `installImage.tag`              | Image tag for the init container that establishes the replica set         | 0.3                                                 |
| `installImage.pullPolicy`       | Image pull policy for the init container that establishes the replica set | IfNotPresent                                        |
| `image.name`                    | MongoDB image name                                                        | mongo                                               |
| `image.tag`                     | MongoDB image tag                                                         | 3.4                                                 |
| `image.pullPolicy`              | MongoDB image pull policy                                                 | IfNotPresent                                        |
| `podAnnotations`                | Annotations to be added to MongoDB pods                                   | {}                                                  |
| `resources`                     | Pod resource requests and limits                                          | {}                                                  |
| `persistentVolume.enabled`      | If `true`, persistent volume claims are created                           | `true`                                              |
| `persistentVolume.storageClass` | Persistent volume storage class                                           | `volume.alpha.kubernetes.io/storage-class: default` |
| `persistentVolume.accessMode`   | Persistent volume access modes                                            | [ReadWriteOnce]                                     |
| `persistentVolume.size`         | Persistent volume size                                                    | 10Gi                                                |
| `persistentVolume.annotations`  | Persistent volume annotations                                             | {}                                                  |
| `auth.enabled`                  | If `true`, keyfile access control is enabled                              | `false`                                             |
| `auth.key`                      | key for internal authentication                                           |                                                     |
| `auth.existingKeySecret`        | If set, an existing secret with this name for the key is used             |                                                     |
| `auth.adminUser`                | MongoDB admin user                                                        |                                                     |
| `auth.adminPassword`            | MongoDB admin password                                                    |                                                     |
| `auth.existingAdminSecret`      | If set, and existing secret with this name is used for the admin user     |                                                     |
| `serviceAnnotations`            | Annotations to be added to the service                                    | {}                                                  |
| `configmap`                     | Content of the MongoDB config file                                        | See below                                           |

*MongoDB config file*

The MongoDB config file `mongod.conf` is configured via the `configmap` configuration value. The defaults from
`values.yaml` are the following:

```yaml
configmap:
  storage:
    dbPath: /data/db
  net:
    port: 27017
  replication:
    replSetName: rs0
```

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`.

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```console
$ helm install tc/mongodb-replicaset --name mongodb-rs --namespace=mongodb \
  -f values.yaml
```

> **Tip**: You can use the default [values.yaml](values.yaml)

Once you have all 3 nodes in running, you can run the "test.sh" script in this directory, which will insert a key into the primary and check the secondaries for output. This script requires that the `$RELEASE_NAME` environment variable be set, in order to access the pods.

## Authentication

By default, this chart creates a MongoDB replica set without authentication. Authentication can be enabled using the
parameter `auth.enabled`. Once enabled, keyfile access control is set up and an admin user with root privileges
is created. User credentials and keyfile may be specified directly. Alternatively, existing secrets may be provided.
The secret for the admin user must contain the keys `user` and `password`, that for the key file must contain `key.txt`.

## Deep dive

Because the pod names are dependent on the name chosen for it, the following examples use the
environment variable `RELEASENAME`. For example, if the helm release name is `mongodb-rs`, one would need to set the following before proceeding. The example scripts below assume 3 pods only.

```console
export RELEASE_NAME=mongodb-rs
```

### Cluster Health

```console
$ for i in 0 1 2; do kubectl exec $RELEASE_NAME-mongodb-replicaset-$i -- sh -c 'mongo --eval="printjson(db.serverStatus())"'; done
```

### Failover

One can check the roles being played by each node by using the following:
```console
$ for i in 0 1 2; do kubectl exec $RELEASE_NAME-mongodb-replicaset-$i -- sh -c 'mongo --eval="printjson(rs.isMaster())"'; done

MongoDB shell version: 3.4.5
connecting to: mongodb://127.0.0.1:27017
MongoDB server version: 3.4.5
{
	"hosts" : [
		"mongodb-rs-mongodb-0.mongodb-rs-mongodb.default.svc.cluster.local:27017",
		"mongodb-rs-mongodb-1.mongodb-rs-mongodb.default.svc.cluster.local:27017",
		"mongodb-rs-mongodb-2.mongodb-rs-mongodb.default.svc.cluster.local:27017"
	],
	"setName" : "rs0",
	"setVersion" : 3,
	"ismaster" : true,
	"secondary" : false,
	"primary" : "mongodb-rs-mongodb-0.mongodb-rs-mongodb.default.svc.cluster.local:27017",
	"me" : "mongodb-rs-mongodb-0.mongodb-rs-mongodb.default.svc.cluster.local:27017",
	"electionId" : ObjectId("7fffffff0000000000000001"),
	"maxBsonObjectSize" : 16777216,
	"maxMessageSizeBytes" : 48000000,
	"maxWriteBatchSize" : 1000,
	"localTime" : ISODate("2016-09-13T01:10:12.680Z"),
	"maxWireVersion" : 4,
	"minWireVersion" : 0,
	"ok" : 1
}


```
This lets us see which member is primary.

Let us now test persistence and failover. First, we insert a key (in the below example, we assume pod 0 is the master):
```console
$ kubectl exec $RELEASE_NAME-mongodb-replicaset-0 -- mongo --eval="printjson(db.test.insert({key1: 'value1'}))"

MongoDB shell version: 3.4.5
connecting to: mongodb://127.0.0.1:27017
{ "nInserted" : 1 }
```

Watch existing members:
```console
$ kubectl run --attach bbox --image=mongo:3.4 --restart=Never --env="RELEASE_NAME=$RELEASE_NAME" -- sh -c 'while true; do for i in 0 1 2; do echo $RELEASE_NAME-mongodb-replicaset-$i $(mongo --host=$RELEASE_NAME-mongodb-replicaset-$i.$RELEASE_NAME-mongodb-replicaset --eval="printjson(rs.isMaster())" | grep primary); sleep 1; done; done';

Waiting for pod default/bbox2 to be running, status is Pending, pod ready: false
If you don't see a command prompt, try pressing enter.
mongodb-rs-mongodb-2 "primary" : "mongodb-rs-mongodb-0.mongodb-rs-mongodb.default.svc.cluster.local:27017",
mongodb-rs-mongodb-0 "primary" : "mongodb-rs-mongodb-0.mongodb-rs-mongodb.default.svc.cluster.local:27017",
mongodb-rs-mongodb-1 "primary" : "mongodb-rs-mongodb-0.mongodb-rs-mongodb.default.svc.cluster.local:27017",
mongodb-rs-mongodb-2 "primary" : "mongodb-rs-mongodb-0.mongodb-rs-mongodb.default.svc.cluster.local:27017",
mongodb-rs-mongodb-0 "primary" : "mongodb-rs-mongodb-0.mongodb-rs-mongodb.default.svc.cluster.local:27017",

```

Kill the primary and watch as a new master getting elected.
```console
$ kubectl delete pod $RELEASE_NAME-mongodb-replicaset-0

pod "mongodb-rs-mongodb-0" deleted
```

Delete all pods and let the statefulset controller bring it up.
```console
$ kubectl delete po -l "app=mongodb-replicaset,release=$RELEASE_NAME"
$ kubectl get po --watch-only
NAME                    READY     STATUS        RESTARTS   AGE
mongodb-rs-mongodb-0   0/1       Pending   0         0s
mongodb-rs-mongodb-0   0/1       Pending   0         0s
mongodb-rs-mongodb-0   0/1       Pending   0         7s
mongodb-rs-mongodb-0   0/1       Init:0/2   0         7s
mongodb-rs-mongodb-0   0/1       Init:1/2   0         27s
mongodb-rs-mongodb-0   0/1       Init:1/2   0         28s
mongodb-rs-mongodb-0   0/1       PodInitializing   0         31s
mongodb-rs-mongodb-0   0/1       Running   0         32s
mongodb-rs-mongodb-0   1/1       Running   0         37s
mongodb-rs-mongodb-1   0/1       Pending   0         0s
mongodb-rs-mongodb-1   0/1       Pending   0         0s
mongodb-rs-mongodb-1   0/1       Init:0/2   0         0s
mongodb-rs-mongodb-1   0/1       Init:1/2   0         20s
mongodb-rs-mongodb-1   0/1       Init:1/2   0         21s
mongodb-rs-mongodb-1   0/1       PodInitializing   0         24s
mongodb-rs-mongodb-1   0/1       Running   0         25s
mongodb-rs-mongodb-1   1/1       Running   0         30s
mongodb-rs-mongodb-2   0/1       Pending   0         0s
mongodb-rs-mongodb-2   0/1       Pending   0         0s
mongodb-rs-mongodb-2   0/1       Init:0/2   0         0s
mongodb-rs-mongodb-2   0/1       Init:1/2   0         21s
mongodb-rs-mongodb-2   0/1       Init:1/2   0         22s
mongodb-rs-mongodb-2   0/1       PodInitializing   0         25s
mongodb-rs-mongodb-2   0/1       Running   0         26s
mongodb-rs-mongodb-2   1/1       Running   0         30s


...
mongodb-rs-mongodb-0 "primary" : "mongodb-rs-mongodb-0.mongodb-rs-mongodb.default.svc.cluster.local:27017",
mongodb-rs-mongodb-1 "primary" : "mongodb-rs-mongodb-0.mongodb-rs-mongodb.default.svc.cluster.local:27017",
mongodb-rs-mongodb-2 "primary" : "mongodb-rs-mongodb-0.mongodb-rs-mongodb.default.svc.cluster.local:27017",
```

Check the previously inserted key:
```console
$ kubectl exec $RELEASE_NAME-mongodb-replicaset-1 -- mongo --eval="rs.slaveOk(); db.test.find({key1:{\$exists:true}}).forEach(printjson)"

MongoDB shell version: 3.4.5
connecting to: mongodb://127.0.0.1:27017
{ "_id" : ObjectId("57b180b1a7311d08f2bfb617"), "key1" : "value1" }
```

### Scaling

Scaling should be managed by `helm upgrade`, which is the recommended way.
