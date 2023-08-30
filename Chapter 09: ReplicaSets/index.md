# ReplicaSets

More often than not, you want multiple replicas of a container running at a particular time for a variety of reasons:
- Redundancy
- Scale
- Sharding

A ReplicaSet acts as a cluster-wide Pod manager, ensuring that the right types and numbers of Pods are running at all times.

The easiest way to think of a ReplicaSet is that it combines a cookie cutter and a desired number of cookies into a single API object. 

The easiest way to think of a ReplicaSet is that it combines a cookie cutter and a desired number of cookies into a single API object.

## Reconciliation Loops

The central concept behind a reconciliation loop is the notion of desired state versus observed or current state. Desired state is the state you want. With a ReplicaSet, it is the desired number of replicas and the definition of the Pod to replicate.

## Relating Pods and ReplicaSets

Decoupling is a key theme in Kubernetes. In particular, it’s important that all of the core concepts of Kubernetes are modular with respect to each other and that they
are swappable and replaceable with other components. In this spirit, the relationship between ReplicaSets and Pods is loosely coupled. Though ReplicaSets create and
manage Pods, they do not own the Pods they create. ReplicaSets use label queries to identify the set of Pods they should be managing. In a similar decoupling, ReplicaSets that create multiple Pods and the services
that load balance to those Pods are also totally separate, decoupled API objects.

### Adopting Existing Containers

Because ReplicaSets are decoupled from the Pods they manage, you can simply create a ReplicaSet that will “adopt” the existing Pod and scale out additional copies of those containers. In this way, you can seamlessly move from a single imperative Pod to a replicated set of Pods managed by a ReplicaSet.

### Quarantining Containers

You can modify the set of labels on the sick Pod. Doing so will disassociate it from the ReplicaSet (and service) so that you can debug the Pod. The ReplicaSet controller will notice that a Pod is missing and create a new copy, but because the Pod is still running, it is available to developers for interactive debugging, which is significantly more valuable than debugging from logs.

## Designing with ReplicaSets

ReplicaSets are designed to represent a single, scalable microservice inside your architecture. heir key characteristic is that every Pod the ReplicaSet controller
creates is entirely homogeneous. Typically, these Pods are then fronted by a Kubernetes service load balancer, which spreads traffic across the Pods that make up the service.

## ReplicaSet Spec

Like all objects in Kubernetes, ReplicaSets are defined using a specification. A spec section that describes the number of Pods (replicas) that should be running cluster-wide at any given time, and a Pod template that describes the Pod to be created when the defined number of replicas is not met.

Example: [Simple ReplicaSet Spec](./kuard-rs.yaml)

### Pod Templates

When the number of Pods in the current state is less than
the number of Pods in the desired state, the ReplicaSet controller will create new Pods using a template contained in the ReplicaSet specification. The Kubernetes ReplicaSet controller creates and submits a Pod manifest based on the Pod template directly to the API server.

### Labels

ReplicaSets monitor cluster state using a set of Pod labels to filter Pod listings and track Pods running within a cluster. When initially created, a ReplicaSet
fetches a Pod listing from the Kubernetes API and filters the results by labels.

### Creating a ReplicaSet

ReplicaSets are created by submitting a ReplicaSet object to the Kubernetes API. 

Use the kubectl apply command to submit the kuard ReplicaSet to the Kubernetes API:

```
$ kubectl apply -f kuard-rs.yaml
replicaset "kuard" created
```

### Inspecting a ReplicaSet

If you are interested in further details about a ReplicaSet, you can use the describe command to provide much more information about its state.

```
$ kubectl describe rs kuard

Name:         kuard
Namespace:    default
Selector:     app=kuard,version=2
Labels:       app=kuard
              version=2
Annotations:  <none>
Replicas:     1 current / 1 desired
Pods Status:  1 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=kuard
           version=2
  Containers:
   kuard:
    Image:        gcr.io/kuar-demo/kuard-amd64:green
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  20s   replicaset-controller  Created pod: kuard-sfjlv

```

### Finding a ReplicaSet from a Pod

Sometimes you may wonder if a Pod is being managed by a ReplicaSet, and if it is, which one. To enable this kind of discovery, the ReplicaSet controller adds an `ownerReferences` section to every Pod that it creates.
If you run the following, look for the ownerReferences section:

```
$ kubectl get pods <pod-name> -o=jsonpath='{.metadata.ownerReferences[0].name}'
```

If applicable, this will list the name of the ReplicaSet that is managing this Pod.

### Finding a Set of Pods for a ReplicaSet

First, get the set of labels using the kubectl describe command. In the previous example, the label selector was app=kuard,version=2.
 To find the Pods that match this selector, use the
`--selector` flag or the shorthand `-l`:

```
$ kubectl get pods -l app=kuard,version=2
```

This is exactly the same query that the ReplicaSetexecutes to determine the current number of Pods.


## Scaling ReplicaSets

You can scale ReplicaSets up or down by updating the `spec.replicas` key on the ReplicaSet object stored in Kubernetes. When you scale up a ReplicaSet, it submits
new Pods to the Kubernetes API using the Pod template defined on the ReplicaSet.

### Imperative Scaling with kubectl scale

The easiest way to achieve this is using the scale command in kubectl. For example, to scale up to four replicas, you could run:

```
$ kubectl scale replicasets kuard --replicas=4
```

## Declaratively Scaling with kubectl apply

In a declarative world, you make changes by editing the configuration file in version control and then applying those changes to your cluster. To scale the kuard Replica‐
Set, edit the kuard-rs.yaml configuration file and set the replicas count to 3:

```
...
spec:
replicas: 3
...
```

### Autoscaling a ReplicaSet

While there will be times when you want to have explicit control over the number of replicas in a ReplicaSet, often you simply want to have “enough” replicas. The definition varies depending on the needs of the containersin the ReplicaSet. For example, with a web server like NGINX, you might want to scale due to CPU usage. For an in-memory cache, you might want to scale with memory consumption. In
some cases, you might want to scale in response to custom application metrics. Kubernetes can handle all of these scenarios via Horizontal Pod Autoscaling (HPA).

```
Autoscaling requires the presence of the metrics-server in your cluster. The metrics-server keeps track of metrics and provides an API for consuming metrics that HPA uses when making scaling decisions.
```

Scaling based on CPU usage is the most common use case for Pod autoscaling. You can also scale based on memory usage. CPU-based autoscaling is most useful for request-based systems that consume CPU proportionally to the number of requests they are receiving, while using a relatively static amount of memory.

To scale a ReplicaSet, you can run a command like the following:
```
$ kubectl autoscale rs kuard --min=2 --max=5 --cpu-percent=80
```

To view, modify, or delete this resource, you can use
the standard kubectl commands:

```
$ kubectl get hpa
```

It’s a bad idea to combine autoscaling with imperative or
declarative management of the number of replicas. If both you and an autoscaler are attempting to modify the number of replicas, it’s highly likely that you will clash, resulting in unexpected behavior.

## Deleting ReplicaSets

When a ReplicaSet is no longer required, it can be deleted using the kubectl delete command. By default, this also deletes the Pods that are managed by the ReplicaSet:

```
$ kubectl delete rs kuard
```

If you don’t want to delete the Pods that the ReplicaSet is managing, you can set the `--cascade` flag to false to ensure only the ReplicaSet object is deleted and not the
Pods:

```
$ kubectl delete rs kuard --cascade=false
```