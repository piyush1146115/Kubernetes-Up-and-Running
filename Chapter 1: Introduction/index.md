# Introduction

People mostly use kubernetes for one of the following benefits:
- Development velocity
- Scaling (of both software and teams)
- Abstracting your infrastructure
- Efficiency
- Cloud native ecosystem

## Velocity

The core concepts that enable this are:
- **Immutability**: Immutable container images are at the core of everything that you will build in
Kubernetes
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