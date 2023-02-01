# Service Meshes

There are three general capabilities provided by most service mesh implementations:
- network encryption and authorization,
- traffic shaping
- observability

### Encryption and Authentication with Mutal TLS

Encryption of network traffic between Pods is a key component to security in a microservice architecture. Encryptions provided by Mutual Transport Layer Security,
or mTLS, is one of the most popular use cases for a service mesh. By contrast, installing a service mesh on your Kubernetes cluster automatically provides
encryption to network traffic between every Pod in the cluster. The service mesh adds a sidecar container to every Pod, which transparently intercepts all network
communication. In addition to securing the communication, mTLS adds identity to the encryption using client certificates so your application securely knows the identity of every network client.

### Traffic Shaping

Experiments are an incredibly useful way to add reliability, agility, and insight to your application, but they are often difficult to implement in practice and thus not used as often as they might otherwise be.

Service meshes change this by building experimentation into the mesh itself. Instead of writing code to implement your experiment, or deploying an entirely new copy of your application on new infrastructure, you declaratively define the parameters of the experiment (10% of traffic to version Y, 90% of traffic to version X), and the service mesh implements it for you. Although, as a developer, you are involved in defining the experiment, the implementation is transparent and automatic, meaning that many more experiments will be run with a corresponding increase in reliability, agility, and insight.

### Introspection

Debugging is even more difficult when applications are spread across multiple microservices. It is hard to stitch together a single request when it spans multiple Pods.

Automatic introspection is another important capability provided by a service mesh. Because it is involved in all communication between Pods, the service mesh knows where requests were routed, and it can keep track of the information required to put a complete request trace back together. Instead of seeing a flurry of requests to a bunch of different microservices, the developer can see a single aggregate request that defines the user experience of their complete application.

## Do You Really Need a Service Mesh?

A service mesh is a distributed system that adds complexity to your application design. The service mesh is deeply integrated into the core communication of your microservices. When a service mesh fails, your entire application stops working. Before you adopt a service mesh, you must be confident that you can fix problems when they occur.

Ultimately, weighing the costs versus benefits of a service mesh is something each application or platform team needs to do at a cluster level. To maximize the benefits of a service mesh, it’s helpful for all microservices in the cluster to adopt it at the same time.

### Introspecting a Service Mesh Implementation

Most service mesh implementations add a sidecar container to every Pod in the mesh. Because the sidecar sits in the same network stack as the application Pod, it can use tools like iptables or, more recently, eBPF to introspect and intercept network traffic coming from your application container and process it into the service mesh. Most service mesh implementations depend on a mutating admission controller to automatically add the service mesh sidecar to all Pods that are created in a particular cluster. Any REST API request to create a Pod is first routed to this admission controller. The service mesh admission controller modifies the Pod definition by adding the sidecar. Because this admission controller is installed by the cluster administrator, it transparently and consistently implements a service mesh for an entire cluster.

But the service mesh isn’t just about modifying the Pod network. You also need to be able to control how the service mesh behaves; for example, by defining routing rules for experiments or access restrictions for services in the mesh. Service mesh implementations take advantages of custom resource definitions (CRDs) to add specialized resources to your Kubernetes cluster that are not part of the default installation. In most cases, the specific custom resources are tightly tied to the service mesh itself. An ongoing effort in the CNCF is defining a standard vendor-neutral Service Mesh Interface (SMI) that many different
service meshes can implement.

### Service Mesh Landscape

The most daunting aspect of the service mesh landscape may be figuring out which mesh to choose. Though
concrete statistics are hard to come by, the most popular service mesh is likely the Istio project. In addition to Istio, there are many other open source meshes, including Linkerd, Consul Connect, Open Service Mesh, and others. There are also proprietary meshes like AWS App Mesh.

How is a developer or cluster administrator to choose? The truth is that the best service mesh for you is likely the one that your cloud provider supplies for you. Adding operating a service mesh to the already complicated duties of your cluster operators is generally unnecessary. It is much better to let a cloud provider manage it for you.

If that isn’t an option for you, do your research. Don’t be drawn in by flashy demos and promises of functionality. A service mesh lives deep within your infrastructure, and any failures can significantly impact the availability of your application. Additionally, because service mesh APIs tend to be implementation specific, it is difficult to change your choice of service mesh once you have spent time developing applications around it. In the end, you may find that the right mesh for you is no mesh at all.