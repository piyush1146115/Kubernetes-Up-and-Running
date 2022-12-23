# Pods

Kubernetes groups multiple containers into a single atomic unit called a Pod. (The name goes with the whale theme of Docker containers, since a pod is also a group of whales.)

## Pods in Kubernetes

A Pod is a collection of application containers and volumes running in the same execution environment. Pods, not containers, are the smallest deployable artifact in a Kubernetes cluster. This means all of the containers in a Pod always land on the same machine.

Each container within a Pod runs in its own cgroup, but they share a number of Linux namespaces.

Applications running in the same Pod share the same IP address and port space(network namespace), have the same hostname (UTS namespace), and can commu‐nicate using native interprocess communication channels over System V IPC or POSIX message queues (IPC namespace).

### Thinking with Pods

In general, the right question to ask yourself when designing Pods is “Will these containers work correctly, if they land on different machines?” If the answer is no, a Pod is the correct grouping for the containers. If the answer is yes, using multiple Pods is probably the correct solution.

### The Pod Manifest

Kubernetes strongly believes in declarative configuration, which means that you write down the desired state of the world in a configuration file and then submit that configuration to a service that takes actions to ensure the desired state becomes the actual state. The Kubernetes API server accepts and processes Pod manifests before storing them in persistent storage (etcd). 

Scheduling multiple replicas of the same application onto the same machine is worse for reliability, since the machine is a single failure domain. Kubernetes scheduler tries to ensure that Pods from the same application are distributed onto different machines for reliability in the presence of such failures. Once scheduled to a node, Pods don’t move and must be explicitly destroyed and rescheduled.

### Creating a pod

The simplest way to create a Pod is via the imperative kubectl run command. For example, to run our same kuard server, use:

```
$ kubectl run kuard --image=gcr.io/kuar-demo/kuard-amd64:blue
```

### Creating a Pod Manifest

Pod manifests include a couple of key fields and attributes: namely, a metadata section for describing the Pod and its labels, a spec section for describing volumes, and a list of containers that will run in the Pod. You can achieve similar result as previous section like below:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:blue
      name: kuard
      ports:
        - containerPort: 8080
        name: http
        protocol: TCP
```

Use the kubectl apply command to launch a single instance of kuard:
```
$ kubectl apply -f kuard-pod.yaml
```

The Pod manifest will be submitted to the Kubernetes API server. The Kubernetes system will then schedule that Pod to run on a healthy node in the cluster, where the kubelet daemon will monitor it.