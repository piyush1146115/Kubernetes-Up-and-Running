# Multicluster Application Deployments

In many cases, a single Kubernetes cluster is tied to a single location and thus is a single failure domain.

In some cases, especially in cloud environments, the Kubernetes cluster is designed to be regional. Regional clusters span across multiple independent zones and are thus resilient to the problems in the underlying infrastructure. In addition to resiliency requirements, another strong driver of multicluster deployments is some business or application need for regional affinity.

## Before You Even Begin

It is critical that you have the right foundations in place in a single cluster deployment before you consider moving to multiple clusters. 

Don’t use a GUI or CLI tool to create your cluster. It may seem cumbersome at first to push all changes through source control and CI/CD, but the stable foundation pays significant dividends.

The same is true of the foundational components that you deploy into your clusters. These components include monitoring, logging, and security scanners, which need to be present before any application is deployed. These tools also need to be managed using infrastructure as code tools like Helm and deployed using automation.

Though Kubernetes supports simple certificate-based authentication, we stronglysuggest using integrations with a global identity provider, such as Azure Active
Directory or any other OpenID Connect–compatible identity provider. Ensuring thateveryone uses the same identity when accessing all of the clusters is a critical part of maintaining security best practices and avoiding dangerous behaviors like sharing certificates.

Just like setting up the right unit testing and build infrastructure is critical to your application development, setting up the right foundation for managing multiple Kubernetes clusters sets the stage for stable application deployments across a broad fleet of infrastructure. In the coming sections, we’ll talk about how to build your application to operate successfully in a multicluster environment.

### Starting at the Top with a Load-Balancing Approach

Access to your application starts with a domain name. This means that the start of your multicluster load-balancing strategy starts with a DNS lookup. This DNS lookup is the first choice in your load-balancing strategy. In many traditional load-balancing approaches, this DNS lookup was used for routing traffic to specific locations. This is generally referred to as “GeoDNS.” In GeoDNS, the IP address returned by the DNS lookup is tied to the physical location of the client. The IP address is generally the regional cluster that is closest to the client.

The other alternative to using DNS to select your cluster is a load-balancing technique known as anycast. With anycast networking, a single static IP address is advertised from multiple locations around the internet using core routing protocols. While traditionally we think of an IP address mapping to a single machine, with anycast networking the IP address is actually a virtual IP address that is routed to a different location depending on your network location. Your traffic is routed to the “closest” location based on the distance in terms of network performance rather than geographic distance.

One final consideration as you design your load balancing is whether the load balancing happens at the TCP or HTTP level. If you are writing an HTTP-based application (as most applications these days are), then using a global HTTP-aware load balancer enables you to be aware of more details of the client communication. For example,you can make load-balancing decisions based on cookies that have been set in the browser.

## Building Applications for Multiple Clusters 

Ideally, your application doesn’t require state, or all of the state is read-only. In such circumstances, there is little that you need to do to support multiple cluster deployments. Your application can be deployed individually to each of your clusters, a load balancer added to the top, and your multicluster deployment is complete.

The challenges of replicated data and customer experience is succinctly captured by this question: “Can I read my own write?” It may seem obvious that the answer should be “Yes,” but achieving this is harder than it seems.

Consistency governs how you think about replicating data. We assume that we want our data to be consistent; that is, that we will be able to read the same data regardless of where we read it from. But the complicating factor is time: how quickly must our data be consistent?

Your choice of concurrency model also has significant implications for your application’s design and is difficult to change. Consequently, choosing your
consistency model is an important first step before designing your application for multiple environments.

### Replicated Silos: The Simplest Cross-Regional Model

The simplest way to replicate your application across multiple clusters and multiple regions is simply to copy your application into every region. Each instance of your application is an exact clone and looks exactly alike no matter which cluster it is running in.

When you design your application this way, each region is its own silo. All of the data that it needs is present within the region, and once a request enters that region, it is served entirely by the containers running in that one cluster. This has significant benefits in terms of reduced complexity, but as is always the case, this comes at the cost of efficiency.

Especially when taking an existing application from single cluster to multicluster, a replicated silos design is the easiest approach to use, but it is worth understanding that it comes with costs that may be sustainable initially but eventually will require your application to be refactored.

### Sharding: Regional Data

Sharding your data across regions means that not all data is present in all of the clusters where your application is present and this (obviously) impacts the design of your application. The routing layer is responsible for determining whether the request needs to go to a local or cross-regional data shard.

### Better Flexibility: Microservice Routing

A better approach is to treat each microservice within your application as a public-facing service in terms of its application design. It may never be expected to actually be public facing, but it should have its own global load balancer. 