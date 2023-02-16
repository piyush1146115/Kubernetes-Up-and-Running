# Policy and Governance for Kubernetes Clusters

Policy is a set of constraints and conditions for how Kubernetes resources can be configured. Governance provides the ability to verify and enforce organizational policies for allresources deployed to a Kubernetes cluster, such as ensuring all resources use current best practices, comply with security policy, or adhere to company conventions.

## Why Policy and Governance Matter

NetworkPolicy allows you to specify what network services and endpoints a Pod can connect to. 

Here are some common examples of policies that cluster administrators often
configure:
- All containers must only come from a specific container registry
- All Pods must be labeled with the department name and contact information
- All Pods must have both CPU and memory resource limits set
- All Ingress hostnames must be unique across a cluster
- A certain service must not be made available on the internet
- Containers must not listen on privileged ports

## Admission Flow

Admission controllers operate inline as an API request flows through the Kubernetes API server and are used to either mutate or validate the API request resource
before it’s saved to storage. Mutating admission controllers allow the resource to be modified; validating admission controllers do not. There are many different types of admission controllers; this chapter focuses on admission webhooks, which are dynamically configurable. They allow a cluster administrator to configure an endpoint to which the API server can send requests for evaluation by creating either a MutatingWebhookConfiguration or a ValidatingWebhookConfiguration resource.

## Policy and Governance with Gatekeeper

Here, we will focus on an open source ecosystem project called [Gatekeeper](https://open-policy-agent.github.io/gatekeeper/website/docs/). Gatekeeper is a Kubernetes-native policy controller that evaluates resources based on defined policy and determines whether to allow a Kubernetes resource to be created or modified. These evaluations happen server-side as the API request flows through the Kubernetes API server, which means each cluster has a single point of processing. Processing the policy evaluations server-side means that you can install Gatekeeper on existing Kubernetes clusters without changing developer tooling, workflows, or continuous delivery pipelines.

Gatekeeper uses custom resource definitions (CRDs) to define a new set of Kubernetes resources specific to configuring it, which allows cluster administrators to use familiar tools like kubectl to operate Gatekeeper. In addition, it provides real-time, meaningful feedback to the user on why a resource was denied and how to remediate the problem. These Gatekeeper-specific custom resources can be stored in source control and managed using GitOps workflows. Gatekeeper also performs resource mutation (resource modification based on defined conditions) and auditing.