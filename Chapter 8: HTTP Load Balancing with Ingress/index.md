# HTTP Load Balancing with Ingress

Kubernetes calls its HTTP-based load-balancing system Ingress. One of the more complex aspects of the pattern is that the user has to manage the load balancer configuration file. In a dynamic environment and as the set of virtual
hosts expands, this can be very complex.

The Kubernetes Ingress system works to simplify this by (a) standardizing that configuration, (b) moving it to a standard
Kubernetes object, and (c) merging multiple Ingress objects into a single config for the load balancer.

The Ingress controller is a software system made up of two parts. The first is the `Ingress proxy`, which is exposed outside the cluster using a service of type: LoadBalancer . This proxy sends requests to “upstream” servers. The other
component is the `Ingress reconciler`, or operator. The Ingress operator is responsible for reading and monitoring Ingress objects in the Kubernetes API and reconfiguring
the Ingress proxy to route traffic as specified in the Ingress resource.

## Ingress Spec Versus Ingress Controllers

While conceptually simple, at an implementation level, Ingress is very different from pretty much every other regular resource object in Kubernetes. Specifically, it is split into a common resource specification and a controller implementation. There is no “standard” Ingress controller that is built into Kubernetes, so the user must install one
of many optional implementations.

### Installing Contour

This is a controller built to configure the open source (and CNCF project) load balancer called Envoy. Envoy is built to be dynamically configured via an API. The Contour Ingress controller takes care of translating the Ingress objects into something that Envoy can understand.

You can install Contour with a simple one-line invocation:
```
$ kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
```
Note that this requires execution by a user who has cluster-admin permissions.

After you install it, you can fetch the external address of Contour via:
```
$ kubectl get -n projectcontour service envoy -o wide
```

### Configuring DNS

To make Ingress work well, you need to configure DNS entries to the external address for your load balancer. You can map multiple hostnames to a single external endpoint and the Ingress controller will direct incoming requests to the appropriate upstream service based on that hostname.

The ExternalDNS project is a cluster add-on that you can use to manage DNS records for you. ExternalDNS monitors your Kubernetes cluster and synchronizes IP addresses for Kubernetes Service resources with an external DNS provider.


## Using Ingress

The simplest way to use Ingress is to have it just blindly pass everything that it sees through to an upstream service.

Apply the [ingress](./simple-ingress.yaml)

Verify that it was set up correctly using kubectl get and kubectl describe :
```
$ kubectl get ingress
$ kubectl describe ingress simple-ingress
```

This sets things up so that any HTTP request that hits the Ingress controller is forwarded on to the alpaca service.

### Using Hostnames

Things start to get interesting when we direct traffic based on properties of the request. The most common example of this is to have the Ingress system look at the
HTTP host header and direct traffic based on that header.

example: [host ingress](./host-ingress.yaml)

### Using Paths

The next interesting scenario is to direct traffic based on not just the hostname, but also the path in the HTTP request. We can do this easily by specifying a path in
the paths entry.

example : [path ingress](./path-ingress.yaml)

When there are multiple paths on the same host listed in the Ingress system, the longest prefix matches. So, in this example, traffic starting with /a/ is forwarded to
the alpaca service, while all other traffic (starting with / ) is directed to the bandicoot service.

## Advanced Ingress Topics and Gotchas

Ingress supports some other fancy features. The level of support for these features differs based on the Ingress controller implementation, and two controllers may implement a feature in slightly different ways.

### Running Multiple Ingress Controllers

There are multiple Ingress controller implementations, and you may want to run multiple Ingress controllers on a single cluster. 

### Running Multiple Ingress Controllers