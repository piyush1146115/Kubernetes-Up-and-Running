# Service Discovery

Kubernetes is a very dynamic system. The API-driven nature of the system encourages others to create higher and higher levels of automation. While the dynamic nature of Kubernetes makes it easy to run a lot of things, it creates problems when it comes to finding those things.

### What Is Service Discovery?

Service-discovery tools help solve the problem of finding which processes are listening at which addresses for which services. A good service-discovery system will enable users to resolve this information quickly and reliably. A good system is also low-latency; clients are updated soon after the information associated with a service changes. Finally, a good service-discovery system can store a richer definition of what that service is. For example, perhaps there are multiple ports associated with the service.

## The Service Object

Real service discovery in Kubernetes starts with a Service object. A Service object is a way to create a named label selector. We can use `kubectl expose` to create a service. To get services in the cluster:

```bash
$ kubectl create deployment alpaca-prod \
--image=gcr.io/kuar-demo/kuard-amd64:blue \
--port=8080
$ kubectl scale deployment alpaca-prod --replicas 3
$ kubectl expose deployment alpaca-prod
$ kubectl get services -o wide
```

The kubernetes service is automatically created for you so that you can find and talk to the Kubernetes API from within the app. The `kubectl expose` command will conveniently pull both the label selector and the relevant ports (8080, in this case) from the deployment definition.

Furthermore, that service is assigned a new type of virtual IP called a cluster IP. This is a special IP address the system will load balance across all of the Pods that are identified by the selector.

### Service DNS

Because the cluster IP is virtual, it is stable, and it is appropriate to give it a DNS address.

Kubernetes provides a DNS service exposed to Pods running in the cluster. This Kubernetes DNS service was installed as a system component when the cluster was first created. The DNS service is, itself, managed by Kubernetes and is a great example
of Kubernetes building on Kubernetes. The Kubernetes DNS service provides DNS names for cluster IPs.

The full DNS name here is alpaca-prod.default.svc.cluster.local.

### Readiness Checks

One nice thing the Service object does is track which of your Pods are ready via a readiness check.

This readiness check is a way for an overloaded or sick server to signal to the system that it doesn’t want to receive traffic anymore. This is a great way to implement graceful shutdown.

### Looking Beyond the Cluster

Oftentimes, the IPs for Pods are only reachable from within the cluster. At some point, we have to allow new traffic in!

The most portable way to do this is to use a feature called NodePorts, which enhance a service even further. In addition to a cluster IP, the system picks a port (or the user can specify one), and every node in the cluster then forwards traffic to that port to the service.

With this feature, if you can reach any node in the cluster, you can contact a service. You can use the NodePort without knowing where any of the Pods for that service are running. This can be integrated with hardware or software load balancers to expose
the service further.

### Load Balancer Integration

Creating a service of type LoadBalancer exposes that service to the public internet.

Edit the alpaca-prod service again ( kubectl edit service alpaca-prod ) and change `spec.type` to LoadBalancer. If you do a `kubectl get services` right away, you’ll see that the EXTERNAL-IP column for alpaca-prod now says <pending>.

You’ll often want to expose your application within only your private network. To achieve this, use an internal load balancer

## Advanced Details

Kubernetes is built to be an extensible system. As such, there are layers that allow for more advanced integrations.

### Endpoints

Some applications (and the system itself) want to be able to use services without using a cluster IP. This is done with another type of object called an Endpoints object.

For every Service object, Kubernetes creates a buddy Endpoints object that contains the IP addresses for that service:

```
$ kubectl describe endpoints alpaca-prod
```

### Manual Service Discovery

Kubernetes services are built on top of `label selectors over Pods`. That means that you can use the Kubernetes API to do rudimentary service discovery without using a Service object at all!

You can always use labels to identify the set of Pods you are interested in, get all of the Pods for those labels, and dig out the IP address. But keeping the correct set of labels to use in sync can be tricky. This is why the Service object was created.


### kube-proxy and Cluster IPs

The kube-proxy watches for new services in the cluster via the API server. It then programs a set of iptables rules in the kernel of that host to rewrite
the destinations of packets so they are directed at one of the endpoints for that service. If the set of endpoints for a service changes (due to Pods coming and going or due to a failed readiness check), the set of iptables rules is rewritten.

The cluster IP itself is usually assigned by the API server as the service is created. However, when creating the service, the user can specify a specific cluster IP. Once set, the cluster IP cannot be modified without deleting and re-creating the Service
object.

### Connecting External Resources to Services Inside a Cluster

You can use a `NodePort` service to expose the service on the IP addresses of the nodes in the cluster. You can then either program a physical load
balancer to serve traffic to those nodes, or use DNS-based load-balancing to spread traffic between the nodes. There are also a variety of open source projects (for example, HashiCorp’s Consul) that can be used to manage connectivity between in-cluster and out-of-cluster resources.