# ibm-redis

[Redis](https://redis.io) is an open source (BSD licensed), in-memory data structure store, used as a database, cache and message broker.

## Introduction

This chart bootstraps a high availability [Redis](https://redis.io) deployment on a [Kubernetes](http://kubernetes.io) cluster using the [Helm](https://helm.sh) package manager.

## Chart Details

The customized Redis server image determines whether the pod that executes it will be a Redis Sentinel,
Master, or Slave and launches the appropriate service. This Helm chart signals Sentinel status with
environment variables. If not set, the newly launched pod will query K8s for an active master. If none
exists, it uses a deterministic means of sensing whether it should launch as master then writes "master"
or "slave" to the label called redis-role as appropriate. It's this label that determines which service a pod
can be seen through.

The redis-role=master pod is the key for the cluster to get started. Sentinels will wait for it to appear
in the service before they finish launching. All other pods wait for the Sentinels to ID the master. Running
Pods also set the labels podIP and runID. runID is the first few characters of the unique run_id value
generated by each Redis sever.

During normal operation, there should be only one redis-role=master pod. If it fails, the Sentinels
will nominate a new master and change all the redis-role values appropriately.

To see the pod roles, run the following:

```shell
kubectl get pods -L redis-role -l release=<releasename>
```

By default this chart installs 1 master, 2 slaves and 3 sentinels.

## Encryption

This chart requires the adopter to encrypt data prior or the cluster owner to enable ipsec

## Prerequisites

- Kubernetes 1.11
- Tiller 2.9.0
- PV support on the underlying infrastructure
- Persistent Volume is required if persistance is enabled. Currently, only volumes created via dynamic provisioning are supported.

## Configuration

The following tables lists the configurable parameters of the Redis chart and their default values.

| Parameter                        | Description                                           | Default                                                   |
| -------------------------------- | ----------------------------------------------------- | --------------------------------------------------------- |
| `arch.amd64`                     | Preference to run on amd64 architecture               | `2 - No preference`                                       |
| `arch.ppc64le`                   | Preference to run on ppc64le architecture             | `2 - No preference`                                       |
| `arch.s390x`                     | Preference to run on s390x architecture               | `2 - No preference`                                       |
| `global.persistence.enabled`                   | Use a PVC to persist data                                                                    | `true`   |
| `global.persistence.useDynamicProvisioning`    | Enables dynamic binding of Persistent Volume Claims to Persistent Volumes                    | `true`   |
| `global.persistence.supplementalGroups`        | Provide the gid of the volumes as list (required for NFS).                                              |
| `global.image.pullSecret`               |Image pull secret that is globally used inside the chart    | ``                                    |
| `global.environmentSize`         | Controls the resource sizes the value can be either 'custom', 'size0' or 'size1'. Size0 is a minimal spec for evaluation or development purposes. The ppc64le options have reduced CPU requests and limits. Set to 'custom' to use sizing set in values.yaml                           | `custom`                                    |
|global.sch.enabled  | Specifies if ibm-sch chart is used as required subchart. If set to false, the umbrella chart has to provide this dependency | `true`|
|global.rbac.create | This global parameter is used when the chart parameter is set invalid value | `true` |
| `global.metering.productName`           | Name of the product that to be used in metering       |`ibm-redis` |
| `global.metering.productID`           | Product ID to be used in metering       |`ibm_redis_1_0_0_Apache_2___77777` |
| `global.metering.productVersion`           | Product version that to be used in metering      |`1.0.0` |
| `global.metering.productMetric`           | Type of metric to be used in metering        |`VIRTUAL_PROCESSOR_CORE` |
| `global.metering.productChargedContainers`         | Product Charted Containers      |`All` |
| `global.metering.cloudpakName`           | Cloud Pak name      |`IBM Cloud Pak for Data` |
| `global.metering.cloudpakId`           | Cloud Pak ID      |`eb9998dcc5d24e3eb5b6fb488f750fe2` |
| `global.metering.cloudpakVersion`           | Cloud Pak version   |`3.0.0` |
| `global.dockerRegistryPrefix`           | Image registry to be gloablly used in the chart                           | `` |
| `global.storageClassName`           | Persistent volume storage class name                           | `` |
| `global.license`           | Change license to true to indicate have read and agree to license agreements : http://ibm.biz/oms-license & http://ibm.biz/oms-apps-license                         | `false` |
| `image.name`                      | Redis image name                                       | `release/opencontent-redis-3`   
| `image.tag`                      | Redis image tag                                       | `1.1.5`                                          |
| `image.pullPolicy`                      | Redis image pull policy                                       | `IfNotPresent`  |
| `creds.image.name`               | creds image repo                                      | `release-master/opencontent-common-utils`         |
| `creds.image.tag`                      | creds image tag                                       | `1.1.5`                                          |
| `creds.image.pullPolicy`               | creds image pull policy                               | `IfNotPresent`                                    |
| `affinity` | Node/pod affinities. If specified replaces the default affinity to run on any amd64 node. | {} |
| `affinityRedis` | Node/pod affinities for Redis server statefulset only. If specified overrides default affinity to run on any amd64 node.| {} |
| `antiAffinity.policy` | Policy for anti affinity of the StatefulSets (both redis server and sentinels). If **soft** is set, the scheduler tries to create the pods on nodes (with the default topology key) not to co-locate them, but it will not be guaranteed. If **hard**, the scheduler tries to create the pods on nodes not to co-locate them and will not create the pods in case of co-location. If the other value, anti affinity is disabled. | `soft` |
| `antiAffinity.topologyKey` | Key for the node label that the system uses to denote a topology domain. | `kubernetes.io/hostname` |
| `auth.authSecretName`            | Provide the secret name if you want to use your own secret with redis password        | ``                      |
| `cachemode`					   | To Deploy redis in cache mode set as `yes`				   | `no`										   |
| `maxmemory`					   | Maximum memory allocation for redis in bytes							   | `100mb`										   |
| `maxmemory_policy`					   | eviction policy after reaching maxmemory							   | `allkeys-lru`										   |
| `appendonly`					   | Enable AOF						   | `yes`										   |
| `timeout`					   | Close the connection after a client is idle for N seconds (0 to disable)							   | `0`										   |
| `tcp_keepalive`					   | Send TCP ACKs to clients every N seconds in absence of communication							   | `300`										   |
| `lua_time_limit`					   | Max execution time of a Lua script in milliseconds							   | `5000`										   |
| `slowlog_log_slower_than`					   | The execution time, in microseconds, that commands must exceed in order to get logged in the slowlog	 | `10000`	|
| `slowlog_max_len`					   | The maximum length of the slow log							   | `120`										   |
| `replicas.servers`               | Number of redis master/slave pods                     | `2`                                                       |
| `replicas.sentinels`             | Number of sentinel pods                               | `3`                                                       |
| `serverService.type`             | Set to "LoadBalancer" to enable access from the VPC   | `ClusterIP`                                               |
| `rbac.create`                    | Create a RBAC resource                                | `true`                                                    |
| `serviceAccount.create`          | Create a service account                              | `true`                                                    |
| `serviceAccount.name`            | Service account name to use. The chart template _fullname_ is used if blank. | _chart template _fullname_        |
| `persistence.accessMode`                    | Use volume as ReadOnly or ReadWrite                                                          | `ReadWriteOnce` |
| `persistence.size`                          | Size of data volume                                                                          | `2Gi`    |
| `dataPVC.name`                              | Prefix that gets the created Persistent Volume Claims                                        | `data`   |
| `dataPVC.selector.label`                    | In case the persistence is enabled and useDynamicProvisioning is disabled the labels can be used to automatically bound persistent volumes claims to precreated persistent volumes. The persistent volumes to be used must have the specified label. Disabled if label is empty. | `` |
| `dataPVC.selector.value`                    | In case the persistence is enabled and useDynamicProvisioning is disabled the labels can be used to automatically bound persistent volumes claims to precreated persistent volumes. The persistent volumes to be used must have label with the specified value.                  | `` |
| `master.livenessProbe.enabled`              | Enable master node liveness probes                                                           | `true`   |
| `master.livenessProbe.initialDelaySeconds`  | Number of seconds after the container has started before the probe is initiated.             | `60`     |
| `master.livenessProbe.periodSeconds`        | How often (in seconds) to perform the probe.                                                 | `10`     |
| `master.livenessProbe.timeoutSeconds`       | Number of seconds after which the probe times out.                                           | `5`      |
| `master.livenessProbe.successThreshold`     | Minimum consecutive successes for the probe to be considered successful after having failed. | `1`      |
| `master.livenessProbe.failureThreshold`     | Number of failures to accept before giving up and marking the pod as Unready.                | `5`      |
| `master.readinessProbe.enabled`             | Enable master node readiness probes                                                          | `true`   |
| `master.readinessProbe.initialDelaySeconds` | Number of seconds after the container has started before the probe is initiated.             | `15`     |
| `master.readinessProbe.periodSeconds`       | How often (in seconds) to perform the probe.                                                 | `10`     |
| `master.readinessProbe.timeoutSeconds`      | Number of seconds after which the probe times out.                                           | `3`      |
| `master.readinessProbe.successThreshold`    | Minimum consecutive successes for the probe to be considered successful after having failed. | `1`      |
| `master.readinessProbe.failureThreshold`    | Number of failures to accept before giving up and marking the pod as Unready.                | `3`      |
| `master.readinessProbe.failureThreshold`    | Number of failures to accept before giving up and marking the pod as Unready.                | `3`      |
| `master.topologySpreadConstraints.enabled`           | Specifies whether the topology spread contraints should be added to Redis server statefulset                                               | `false`                                                       |
| `master.topologySpreadConstraints.maxSkew`           | How much the availability zones can differ in number of etcd pods.                                                                         | `1`                                                           |
| `master.topologySpreadConstraints.topologyKey`       | Label of nodes defining availability/failure zone                                                                                          | `failure-domain.beta.kubernetes.io/zone`                      |
| `master.topologySpreadConstraints.whenUnsatisfiable` | Action in case new pod cannot be scheduled because of topology contraints.                                                                 | `ScheduleAnyway`                                              |
| `sentinel.livenessProbe.enabled`              | Enable sentinel node liveness probes                                                         | `true`   |
| `sentinel.livenessProbe.initialDelaySeconds`  | Number of seconds after the container has started before the probe is initiated.             | `60`     |
| `sentinel.livenessProbe.periodSeconds`        | How often (in seconds) to perform the probe.                                                 | `10`     |
| `sentinel.livenessProbe.timeoutSeconds`       | Number of seconds after which the probe times out.                                           | `5`      |
| `sentinel.livenessProbe.successThreshold`     | Minimum consecutive successes for the probe to be considered successful after having failed. | `1`      |
| `sentinel.livenessProbe.failureThreshold`     | Number of failures to accept before giving up and marking the pod as Unready.                | `5`      |
| `sentinel.readinessProbe.enabled`             | Enable sentinel node readiness probes                                                        | `true`   |
| `sentinel.readinessProbe.initialDelaySeconds` | Number of seconds after the container has started before the probe is initiated.             | `15`     |
| `sentinel.readinessProbe.periodSeconds`       | How often (in seconds) to perform the probe.                                                 | `10`     |
| `sentinel.readinessProbe.timeoutSeconds`      | Number of seconds after which the probe times out.                                           | `3`      |
| `sentinel.readinessProbe.successThreshold`    | Minimum consecutive successes for the probe to be considered successful after having failed. | `1`      |
| `sentinel.readinessProbe.failureThreshold`    | Number of failures to accept before giving up and marking the pod as Unready.                | `3`      |
| `sentinel.topologySpreadConstraints.enabled`           | Specifies whether the topology spread contraints should be added to sentinel statefulset                                                   | `false`                                                       |
| `sentinel.topologySpreadConstraints.maxSkew`           | How much the availability zones can differ in number of etcd pods.                                                                         | `1`                                                           |
| `sentinel.topologySpreadConstraints.topologyKey`       | Label of nodes defining availability/failure zone                                                                                          | `failure-domain.beta.kubernetes.io/zone`                      |
| `sentinel.topologySpreadConstraints.whenUnsatisfiable` | Action in case new pod cannot be scheduled because of topology contraints.                                                                 | `ScheduleAnyway`                                              |
| `resources.server.requests.memory`   | Server memory resource requests                     | `200Mi`        |
| `resources.server.requests.cpu`      | Server CPU resource requests                        | `100m`         |
| `resources.server.limits.memory`     | Server Memory resource limits                       | `700Mi`        |
| `resources.server.limits.cpu`        | Server CPU resource limits                          | `100m`         |
| `resources.sentinel.requests.memory` | Sentinel memory resource requests                   | `200Mi`        |
| `resources.sentinel.requests.cpu`    | Sentinel CPU resource requests                      | `100m`         |
| `resources.sentinel.limits.memory`   | Sentinel Memory resource limits                     | `200Mi`        |
| `resources.sentinel.limits.cpu`      | Sentinel CPU resource limits                        | `100m`         |
| `securityContext.redis.runAsUser`    | The User ID that needs to be run as by all redis containers. This applies only when installed on non-openshift clusters.      | `1000`   |
| `securityContext.redis.runAsGroup`   | The Group ID that needs to be run as by all redis containers. This applies only when installed on non-openshift clusters.     | `1000`   |
| `securityContext.redis.fsGroup`      | The FS Group ID that needs to be run as by all redis containers. This applies only when installed on non-openshift clusters.  | `1000`   |
| `securityContext.creds.runAsUser`    | The User ID that needs to be run as by all creds job containers. This applies only when installed on non-openshift clusters.  | `99`     |
| `metering`                           | Metering annotations                                                                                                          | `{}`     |


Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,


The above command sets the Redis server within  `default` namespace.

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,


> **Tip**: You can use the default values.yaml

## Features

- Supports redis version 3.2.12 only with the mentioned image
- Supports Authentication
- Supports High Availability
- Supports Redis Cache Mode
- Supports Configuration Paramters to be passed while installing
- Supports to restore data from rdb file

## Limitations

- No SSL support as Redis does not support SSL
- The chart does not support the s390x architecture.
- No support for Hostpath, Gluster, NFS, or Network attached storage

## Resources Required

By Default the chart deploys pods consuming minimum resources as specified in the resources configuration parameter (default: Memory: 200Mi, CPU: 100m)
If using envrionmentSize settings refer to the table below:

| Size     | Memory       | CPU      | Replication  |
| -------- | ------------ | -------- | ------------ |
| Size 0   | 200Mi        | 50m      | 1            |
| Size 1   | 350Mi        | 200m     | 3            |

## Known Issues

At times when the master pod restarts, the slave pod is not correctly labeled as the new master, resulting in the following possible scenarios
- both the pods get labeled as slaves
- both the pods get labeled as master

## PodSecurityPolicy Requirements

The predefined PodSecurityPolicy name [`ibm-restricted-psp`](https://ibm.biz/cpkspec-psp) has been verified for this chart. If your target namespace is bound to this PodSecurityPolicy, you can proceed to install the chart.

This chart also defines a custom PodSecurityPolicy which can be used to finely control the permissions/capabilities needed to deploy this chart. You can enable this custom PodSecurityPolicy using the user interface or the supplied instructions/scripts in the pak_extension pre-install directory.

- From the user interface, you can copy and paste the following snippets to enable the custom PodSecurityPolicy
  - Custom PodSecurityPolicy definition:

    ```yaml
    apiVersion: extensions/v1beta1
    kind: PodSecurityPolicy
    metadata:
      name: ibm-redis-psp
    spec:
      allowPrivilegeEscalation: false
      forbiddenSysctls:
      - '*'
      fsGroup:
        ranges:
        - max: 65535
          min: 1
        rule: MustRunAs
      requiredDropCapabilities:
      - ALL
      runAsUser:
        rule: MustRunAsNonRoot
      seLinux:
        rule: RunAsAny
      supplementalGroups:
        ranges:
        - max: 65535
          min: 1
        rule: MustRunAs
      volumes:
      - configMap
      - emptyDir
      - projected
      - secret
      - downwardAPI
      - persistentVolumeClaim
    ```

  - Custom ClusterRole for the custom PodSecurityPolicy:

    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: ibm-redis-psp
    rules:
    - apiGroups:
      - extensions
      resourceNames:
      - ibm-chart-dev-psp
      resources:
      - podsecuritypolicies
      verbs:
      - use
    ```

## Red Hat OpenShift SecurityContextConstraints Requirements

This chart requires a `SecurityContextConstraints` to be bound to the target namespace prior to installation. To meet this requirement there may be cluster scoped as well as namespace scoped pre and post actions that need to occur.

The predefined `SecurityContextConstraints` name: [`restricted`](https://ibm.biz/cpkspec-scc) has been verified for this chart, if your target namespace is bound to this `SecurityContextConstraints` resource you can proceed to install the chart.

- From the user interface, you can copy and paste the following snippets to enable the custom `SecurityContextConstraints`
  - Custom SecurityContextConstraints definition:

    ```yaml
    apiVersion: security.openshift.io/v1
    kind: SecurityContextConstraints
    metadata:
      name: ibm-chart-dev-scc
    readOnlyRootFilesystem: false
    allowedCapabilities:
    - CHOWN
    - DAC_OVERRIDE
    - SETGID
    - SETUID
    - NET_BIND_SERVICE
    seLinux:
      type: RunAsAny
    supplementalGroups:
      type: RunAsAny
    runAsUser:
      type: RunAsAny
    fsGroup:
      rule: RunAsAny
    volumes:
    - configMap
    - secret
    ```

## Installing the Chart

Clone this repo and execute the below command from the root directory of it.

```shell
helm install .  --name <releasename> --set global.license=true --set global.dockerRegistryPrefix=<DockerRegistry> --set global.image.pullSecret=<docker registry secret> --tls
```

or

```shell
helm install ibm-redis-1.4.11.tgz --name <releasename> --set global.license=true --set global.dockerRegistryPrefix=<DockerRegistry> --set global.image.pullSecret=<docker registry secret> --tls
```

## Verifying the Chart

- To list the resources created by this chart use the flag
`-l release=<releasename>`

- To list the redis-role of the pods deployed by this chart use the below command

```shell
kubectl get pods -l release=<releasename> -L redis-role
```

- To get the password

```shell
kubectl get secret <releasename>-ibm-redis-authsecret  -o json | jq -r .data.password | base64 -D
```

- To test connection execute shell in any of the redis containers in the same cluster as the redis deployment

```shell
kubectl exec -it <podname> bash

$ redis-cli -h  <releasename>-ibm-redis-master-svc -a <password>

> set <key> <value>
> get <key>
```

## Uninstalling the Chart

```shell
helm delete <releasename> --purge --tls
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Backup & Restore

**To Backup:**

dump.rdb is updated periodically, to get the current data in your backup save it before taking backup.
```
$ redis-cli -h  <releasename>-ibm-redis-master-svc -a <password>

> save
```

Simply copy the dump.rdb file from redis master pod and store it in your desired place.

```shell
kubectl cp <namespace>/<podname>:/home/appuser/redis-master-data/dump.rdb local/path/dump.rdb
```

**To Restore:**

Restore data in a new deployment, if not the data available before the restoration process in the deployment will be lost after the restoration process.

1. Need to create a new redis deployment with the following configuration.

- Persistent Volume enabled.
- AOF disabled (set parameter `appendonly no` )

2. Copy the dump file to all of your redis server pods that belongs to that release

```shell
kubectl cp local/path/dump.rdb <namespace>/<podname>:/home/appuser/redis-master-data/dump.rdb
```

3. After copying the dump file to each redis server pods that belongs to that release, Delete all the redis server pods

```shell
kubectl delete pod <podname>
```

4. Wait till all the redis pods are recreated and then connect to redis master service

```shell
$ redis-cli -h  <releasename>-ibm-redis-master-svc -a <password>

> keys *
```

  verify the data is restored

5. Enable AOF

```shell
> config set appendonly yes
> config get appendonly
```
