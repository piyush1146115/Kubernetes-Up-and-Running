# DaemonSets

A DaemonSet ensures that a copy of a Pod is running across a set of nodes in a Kubernetes cluster. DaemonSets are used to deploy system daemons such as log collectors and monitoring agents, which typically must run on every node. 


DaemonSets share similar functionality with ReplicaSets; both create Pods that are expected to be long-running services and ensure that the desired state and the observed state of the cluster match. ReplicaSets should be used when your application is completely decoupled from the node and you can run multiple copies on a given node without special consideration. DaemonSets should be used when a single copy of your application must run on all or a subset of the nodes in the cluster.

You can use labels to run DaemonSet Pods on specific nodes; for example, you may want to run specialized intrusion-detection software on nodes that are exposed to the edge network. You can also use DaemonSets to install software on nodes in a cloud-based cluster.

## DaemonSet Scheduler

By default, a DaemonSet will create a copy of a Pod on every node unless a node selector is used, which will limit eligible nodes to those with a matching set of labels.

Like ReplicaSets, DaemonSets are managed by a reconciliation control loop that measures the desired state (a Pod is present on all nodes) with the observed state (is the Pod present on a particular node?). Given this information, the DaemonSet controller creates a Pod on each node that doesn’t currently have a matching Pod.

If a new node is added to the cluster, then the DaemonSet controller notices that it is missing a Pod and adds the Pod to the new node.

## Creating DaemonSets

DaemonSets are created by submitting a DaemonSet configuration to the Kubernetes API server. The DaemonSet in the following Example  will create a fluentd logging agent on every node in the target cluster.

[fluentd-daemonSets](./fluentd.yaml)

DaemonSets require a unique name across all DaemonSets in a given Kubernetes namespace. Each DaemonSet must include a Pod template spec, which will be used to create Pods as needed. This is where the similarities between ReplicaSets and DaemonSets end. Unlike ReplicaSets, DaemonSets will create Pods on every node in the cluster by default unless a node selector is used.

Once the fluentd DaemonSet has been successfully submitted to the Kubernetes API, you can query its current state using the kubectl describe command:
```
$ kubectl describe daemonset fluentd
```

This output indicates a fluentd Pod was successfully deployed to all three nodes in our cluster. We can verify this using the kubectl get pods command with the -o flag
to print the nodes where each fluentd Pod was assigned:

```
$ kubectl get pods -l app=fluentd -o wide
```

## Limiting DaemonSets to Specific Nodes

There are some cases where you want to deploy a Pod
to only a subset of nodes. For example, maybe you have a workload that requires a GPU or access to fast storage only available on a subset of nodes in your cluster. In
cases like these, node labels can be used to tag specific nodes that meet workload requirements.

### Adding Labels to Nodes

The first step in limiting DaemonSets to specific nodes is to add the desired set of labels to a subset of nodes. This can be achieved using the kubectl label command.

The following command adds the ssd=true label to a single node:
```
$ kubectl label nodes k0-default-pool-35609c18-z7tb ssd=true
```

Using a label selector, we can filter nodes based on labels.
```
$ kubectl get nodes --selector ssd=true
```

### Node Selectors

Node selectors can be used to limit what nodes a Pod can run on in a given Kubernetes cluster. Node selectors are defined as part of the Pod spec when creating a DaemonSet. The DaemonSet configuration in  the following Example limits NGINX to running only on nodes with the ssd=true label set.

Example: [nginx-fast-storage-damonset](./nginx-fast-storage.yaml)

Adding the ssd=true label to additional nodes will cause the nginx-fast-storage Pod to be deployed on those nodes. The inverse is also true: if a required label is
removed from a node, the Pod will be removed by the DaemonSet controller.

## Updating a DaamonSet

DaemonSets can be rolled out using the same RollingUpdate strategy that Deployments use. You can configure the update strategy using the `spec.update Strategy.type` field, which should have the value RollingUpdate.

There are two parameters that control the rolling update of a DaemonSet:
`spec.minReadySeconds` Determines how long a Pod must be “ready” before the rolling update proceeds to upgrade subsequent Pods

`spec.updateStrategy.rollingUpdate.maxUnavailable`
Indicates how many Pods may be simultaneously updated by the rolling update

Once a rolling update has started, you can use the kubectl rollout commands to see the current status of a DaemonSet rollout. For example, `kubectl rollout status daemonSets my-daemon-set` will show the current rollout status of a DaemonSet named my-daemon-set.

## Deleting a DaemonSet

Deleting a DaemonSet using the kubectl delete command is pretty straightfoward. Just be sure to supply the correct name of the DaemonSet you would like to delete:

```
$ kubectl delete -f fluentd.yaml
```