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

If you have a cluster that is configured to integrate with external load balancers, you can use the LoadBalancer type. This builds on the NodePort type by additionally configuring the cloud to create a new load balancer and direct it at nodes in your cluster..

Creating a service of type LoadBalancer exposes that service to the public internet. Before you do this, you should make certain that it is something that is secure to be exposed to everyone in the world.