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

## Creating a ReplicaSet