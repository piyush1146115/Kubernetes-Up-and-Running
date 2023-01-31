# Service Meshes

There are three general capabilities provided by most service mesh implementations:
- network encryption and authorization,
- traffic shaping
- observability

## Encryption and Authentication with Mutal TLS

Encryption of network traffic between Pods is a key component to security in a microservice architecture. Encryptions provided by Mutual Transport Layer Security,
or mTLS, is one of the most popular use cases for a service mesh. By contrast, installing a service mesh on your Kubernetes cluster automatically provides
encryption to network traffic between every Pod in the cluster. The service mesh adds a sidecar container to every Pod, which transparently intercepts all network
communication. In addition to securing the communication, mTLS adds identity to the encryption using client certificates so your application securely knows the identity of every network client.

## Traffic Shaping

Experiments are an incredibly useful way to add reliability, agility, and insight to your application, but they are often difficult to implement in practice and thus not used as often as they might otherwise be.

Service meshes change this by building experimentation into the mesh itself. Instead of writing code to implement your experiment, or deploying an entirely new copy
of your application on new infrastructure, you declaratively define the parameters of the experiment (10% of traffic to version Y, 90% of traffic to version X), and the service mesh implements it for you. Although, as a developer, you are involved in defining the experiment, the implementation is transparent and automatic, meaning that many more experiments will be run with a corresponding increase in reliability, agility, and insight.

## Introspection

Debugging is even more difficult when applications are spread across multiple microservices. It is hard to stitch together a single request when it spans multiple Pods.

Automatic introspection is another important capability provided by a service mesh.Because it is involved in all communication between Pods, the service mesh knows
where requests were routed, and it can keep track of the information required to put
a complete request trace back together. Instead of seeing a flurry of requests to a
bunch of different microservices, the developer can see a single aggregate request that defines the user experience of their complete application.