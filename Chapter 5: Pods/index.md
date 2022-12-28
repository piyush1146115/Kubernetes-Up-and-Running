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

### Listing Pods

Using the kubectl command-line tool, we can list all Pods running in the cluster.

```
kubectl get pods
```

If you ran this command immediately after the Pod was created, you might see:

```
NAME   READY  STATUS   RESTARTS  AGE
kuard  0/1    pending  0         1s
```

The Pending state indicates that the Pod has been submitted but hasn’t been scheduled yet. If a more significant error occurs, such as an attempt to create a Pod with a container image that doesn’t exist, it will also be listed in the status field.

Adding `-o wide` to any kubectl command will print out
slightly more information (while still keeping the information to a single line). Adding `-o json` or `-o yaml` will print out the complete objects in JSON or YAML, respectively.

### Pod Details

To find out more information about a Pod (or any Kubernetes object), you can use the kubectl describe command. For example, to describe the Pod we previously created, you can run:

```
$ kubectl describe pods kuard
```

### Deleting a Pod

When it is time to delete a Pod, you can delete it either by name:
```
$ kubectl delete pods/kuard
```
or you can use the same file that you used to create it:
```
$ kubectl delete -f kuard-pod.yaml
```

All Pods have a termination grace period. By default, this is 30 seconds. When a Pod is transitioned
to Terminating , it no longer receives new requests. In a serving scenario, the grace period is important for reliability because it allows the Pod to finish any active requests that it may be in the middle of processing before it is terminated.


## Accessing Your Pod

Now that your Pod is running, you’re going to want to access it for a variety of reasons.

### Getting More Information with Logs

The kubectl logs command downloads the current logs from the running instance:

```
$ kubectl logs kuard
```

Adding the `-f` flag will cause the logs to stream continuously.

### Running Commands in Your Container with `exec`

To do this, you can use:
```
$ kubectl exec kuard -- date
```
You can also get an interactive session by adding the 
`-it` flag:
```
$ kubectl exec -it kuard -- ash
```