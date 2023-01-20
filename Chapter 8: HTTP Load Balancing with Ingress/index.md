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

You may want to run multiple Ingress controllers on a single cluster. To solve this case, the `IngressClass`
resource exists so that an Ingress resource can request a particular implementation.

If you specify multiple Ingress objects, the Ingress controllers should read them all and try to merge them into a coherent configuration. However, if you specify
duplicate and conflicting configurations, the behavior is undefined.

### Ingress and Namespaces

First, due to an abundance of security caution, an Ingress object can refer to only an upstream service
in the same namespace. This means that you can’t use an Ingress object to point a subpath to a service in another namespace.

### Path Rewriting

Some Ingress controller implementations support optionally, doing path rewriting. This can be used to modify the path in the HTTP request as it gets proxied.

There are multiple implementations that not only implement path rewriting, but also support regular expressions when specifying the path. For example, the NGINX controller allows regular expressions to capture parts of the path and then use that captured content when doing rewriting.

### Serving TLS

When serving websites, it is becoming increasingly necessary to do so securely using TLS and HTTPS. Ingress supports this (as do most Ingress controllers).

First, users need to specify a Secret with their TLS certificate and keys. Once you have the certificate uploaded, you can reference it in an Ingress object. This
specifies a list of certificates along with the hostnames that those certificates should be used for. Again, if multiple Ingress objects specify certificates for
the same hostname, the behavior is undefined.

### Alternate Ingress Implementations

There are many different implementations of Ingress controllers, each building on the base Ingress object with unique features. It is a vibrant ecosystem.

First, each cloud provider has an Ingress implementation that exposes the specific cloud-based L7 load balancer for that cloud.

The most popular generic Ingress controller is probably the open source NGINX Ingress controller.

[Emissary](https://github.com/emissary-ingress/emissary) and [Gloo](https://github.com/solo-io/gloo) are two other Envoy-based Ingress controllers that are focused on being API gateways.

[Traefik](https://doc.traefik.io/traefik/) is a reverse proxy implemented in Go that also can function as an Ingress controller. It has a set of features and dashboards that are very developer-friendly.

## The Future of Ingress

As you have seen, the Ingress object provides a very useful abstraction for configuring L7 load balancers—but it hasn’t scaled to all the features that users want and
various implementations are looking to offer.

The future of HTTP load balancing for Kubernetes looks to be the Gateway API, which is in the midst of development by the Kubernetes special interest group (SIG)
dedicated to networking. The Gateway API project is intended to develop a more modern API for routing in Kubernetes. Though it is more focused on HTTP balanc‐
ing, Gateway also includes resources for controlling Layer 4 (TCP) balancing.