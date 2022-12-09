# Introduction

People mostly use kubernetes for one of the following benefits:
- Development velocity
- Scaling (of both software and teams)
- Abstracting your infrastructure
- Efficiency
- Cloud native ecosystem

## Velocity

The core concepts that enable this are:
- **Immutability**: Immutable container images are at the core of everything that you will build in Kubernetes
- **Declarative configuration** : While imperative commands define actions, declarative configurations define state.
- **Online self-healing systems**: Kubernetes continuously takes actions to ensure that the current state matches the desired state.
- Shared reusable libraries and tools

## Scaling Your Service and Your Teams

Kubernetes achieves scalability by favoring decoupled architectures.

- **Decoupling**: In a decoupled architecture, each component is separated from other components by defined APIs and service load balancers.
- **Easy Scaling for Applications and Clusters**: Because your containers are immutable, and the number of replicas is merely a number in a declarative config,scaling your service upward is simply a matter of changing a number in a configura‐tion file, asserting this new declarative state to Kubernetes, and letting it take care of the rest. Kubernetes can simplify forecasting future compute costs.
- **Scaling Development Teams with Microservices**: Kubernetes provides numerous abstractions and APIs that make it easier to build these decoupled microservice architectures: Pods, Kubernetes services, Namespaces, Ingress. Decoupling the application container image and machine means that different microservices can colocate on the same machine without interfering with one another, reducing the overhead and cost of microservice architectures.
- **Separation of Concerns for Consistency and Scaling**: The decoupling and separation of concerns produced by the Kubernetes stack lead to significantly greater consistency for the lower levels of your infrastructure.

## Abstracting Your Infrastructure

The move to application-oriented container APIs like Kubernetes has two concrete benefits. First, it separates developers from specific machines.
Kubernetes has a number of plug-ins that can abstract you from a particular cloud.

## Efficiency

- Kubernetes provides tools that automate the distribution of applications across a cluster of machines, ensuring higher levels of utilization than are possible with traditional tooling.
- A further increase in efficiency comes from the fact that a developer’s test environ‐ment can be quickly and cheaply created as a set of containers running in a personal view of a shared Kubernetes cluster (using a feature called namespaces).

## Cloud Native Ecosystem

Following the lead of Kubernetes (and Docker and Linux before it), most of these projects are also open source. This means that a developer beginning to build does not have to start from scratch. In the years since it was released, tools for nearly every task, from machine learning to continuous development and serverless programming models have been built for Kubernetes. Indeed, in many cases the challenge isn’t finding a potential solution, but rather deciding which of the many solutions is best suited to the task. The wealth of tools in the cloud native ecosystem has itself become a strong reason for many people to adopt Kubernetes.