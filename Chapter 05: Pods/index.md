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

## Health Checks

Kubernetes introduced health checks for application liveness.

**Liveness Probe**: Liveness probes are defined per container, which means each container inside a Pod is health checked separately. Containers that fail liveness checks are restarted.

**Readiness Probe**: Readiness describes when a container is ready to serve user requests. Containers that fail readiness checks are removed from service load balancers. Readiness probes are configured similarly to liveness probes.

**Startup Probe**: When a Pod is started, the startup probe is run before any other probing of the Pod is started. The startup probe proceeds until it either times out (in which case the Pod is restarted) or it succeeds, at which time the
liveness probe takes over.

**Advanced Probe Configuration**: Probes in Kubernetes have a number of advanced options, including how long to wait
after Pod startup to start probing, how many failures should be considered a true failure, and how many successes are necessary to reset the failure count.

**Other Types of Health Checks**: In addition to HTTP checks, Kubernetes also supports tcpSocket health checks that
open a TCP socket; if the connection succeeds, the probe succeeds. This style of probe is useful for non-HTTP applications, such as databases or other non–HTTP-
based APIs.

## Resource Management

Kubernetes allows users to specify two different resource metrics. `Resource requests` specify the minimum amount of a resource required to run the application.
`Resource limits specify the maximum amount of a resource that an application can consume.

### Resource Requests: Minimum Required Resources

When a Pod requests the resources required to run its containers, Kubernetes guar‐
antees that these resources are available to the Pod.

[example of resource requests](./kuard-pod-resreq.yaml)

Resources are requested per container, not per Pod. The total resources requested by the Pod is the sum of all resources requested by all containers in the Pod because the different containers often have very different CPU requirements.

CPU requests are implemented using the cpu-shares functionality in the Linux kernel.

If a container is over its memory request, the
OS can’t just remove memory from the process, because it’s been allocated. Consequently, when the system runs out of memory, the kubelet terminates containers whose memory usage is greater than their requested memory. These containers are automatically
restarted, but with less available memory on the machine for the container to consume.

### Capping Resource Usage with Limits

In addition to setting the resources required by a Pod, which establishes the mini‐
mum resources available to it, you can also set a maximum on a its resource usage via
resource limits.

[example of resource limits](./kuard-pod-reslim.yaml)

When you establish limits on a container, the kernel is configured to ensure that
consumption cannot exceed these limits. A container with a CPU limit of 0.5 cores will only ever get 0.5 cores, even if the CPU is otherwise idle. A container with a memory limit of 256 MB will not be allowed additional memory; for example, malloc will fail if its memory usage exceeds 256 MB.

## Persisting Data with Volumes

In some use cases, having access to persistent disk storage is an important part of a healthy application. Kubernetes models such persistent storage.

### Using Volumes with Pods

To add a volume to a Pod manifest, there are two new stanzas to add to our configuration. The first is a new `spec.volumes` section. This array defines all of the volumes that may be accessed by containers in the Pod manifest. It’s important to note that not all containers are required to mount all volumes defined in the Pod. The second addition is the `volumeMounts` array in the container definition. This array defines
the volumes that are mounted into a particular container and the path where each volume should be mounted. Note that two different containers in a Pod can mount the same volume at different mount paths.

[example: pod manifest with volume](./kuard-pod-vol.yaml)

### Different Ways of Using Volumes with Pods

- Communication/synchronization
- Cache
- Persistent data
- Mounting the host filesystem