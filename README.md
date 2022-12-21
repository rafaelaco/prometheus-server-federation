# Prometheus Federation

This chart creates the [Prometheus Federation](https://prometheus.io/docs/prometheus/latest/federation/) structure, making possible to have multiple [Prometheus](https://prometheus.io/) instances running in the same kubernetes cluster.

## Prerequisites

- Helm 3.7+

## Test

Just run the helm template command to see the generates elements.

```console
helm template prom .
```

## Configuration

Open the [values.yaml](./values.yaml) file to see all the configuration options.

### Basic primary/replica setup

```yaml
  prometheusInstances:
    - type: primary
      storageTsdbRetentionSize: "5d"

    - type: replica
      namespaceToScrape: [kube-system]
      labelToScrape: "k8s-app=kube-dns"
      storageTsdbRetentionSize: "1d"

    - type: replica
      namespaceToScrape: [microservices]
      shards: 5
      storageTsdbRetentionSize: "1d"
```

### RBAC

For everything to work as expected we create ClusterRoles that will provide the following permissions to the prometheus instances:

```yaml
  rules:
    - apiGroups: [""]
      resources:
        - nodes
        - services
        - endpoints
        - pods
      verbs: ["get", "list", "watch"]
    - apiGroups:
        - extensions
      resources:
        - ingresses
      verbs: ["get", "list", "watch"]
```

You can see details opening the file [cluster-role.yaml](./templates/cluster-role.yaml).

### ConfigMap

The prometheus configuration resides in the configmaps, you can check the details opening the file [configmap.yaml](./templates/configmap.yaml).

### Persistent Volume

This Charts was created to the Azure Kubernetes environment, because of that we use Azure CSI as Persistent Volume.