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

### What Is Open Policy Agent?

At the core of Gatekeeper is [Open Policy Agent](https://www.openpolicyagent.org/), a cloud native open source policy engine that is extensible and allows policy to be portable across different applications. Open Policy Agent (OPA) is responsible for performing all policy evaluations and returning either an admit or deny. This gives Gatekeeper access to an ecosystem of policy tooling, such as, [Conftest](https://github.com/open-policy-agent/conftest), which enables you to write policy tests and implement them in continuous integration pipelines before deployment.

Open Policy Agent exclusively uses a native query language called Rego for all policies.

### Installing Gatekeeper

Before you start configuring policies, you’ll need to install Gatekeeper. Gatekeeper components run as Pods in the gatekeeper-system namespace and configure a
webhook admission controller.

```
$ helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts
$ helm install gatekeeper/gatekeeper --name-template=gatekeeper \
--namespace gatekeeper-system --create-namespace
```

### Configuring Policies

First, you’ll need to configure the policy we need to create a custom resource called a constraint template. This is usually done by a cluster administrator. Now you can create a constraint resource to put the policy into effect (again, playing the role of the cluster administrator).

### Understanding Constraint Templates

This constraint template has an apiVersion and kind that are part of the custom resources used only by Gatekeeper. Under the spec section, you’ll see the name
K8sAllowedRepos : remember that name, because you’ll use it as the constraint kind when creating constraints.

### Creating Constraints

Let’s take a closer look at the constraint in the [Example](./allowedrepos-constraint.yaml), which allows only container images that originate from gcr.io/kuar-demo/. You may notice that the constraint is of the kind “K8sAllowedRepos,” which was defined as part of the constraint template. It also defines an enforcementAction of “deny,” meaning that noncompliant resources will be denied. The match portion defines the scope of this constraint, all Pods in the default namespace. Finally, the parameters section is required to satisfy the constraint template (an array of strings).

### Audit

Gatekeeper’s audit capabilities allow cluster administrators to get a list of current, noncompliant resources on a cluster. To audit the list of noncompliant resources for a given constraint, run a kubectl get constraint on that constraint and specify that you want the output in YAML format as follows:

```
$ kubectl get constraint repo-is-kuar-demo -o yaml
```

Under the status section, you can see the auditTimestamp , which is the last time the audit was run. totalViolations lists the number of resources that violate this constraint. The violations section lists the violations. We can see that the nginx-noncompliant Pod is in violation and the message with the details why. Using a constraint enforcementAction of “dryrun” along withaudit is a powerful way to confirm that your policy is having the
desired impact. It also creates a workflow to bring resources into compliance.

### Mutation

By default, Gatekeeper is only deployed as a validating admission webhook, but it can be configured to operate as a mutating admission webhook. To enable mutation, please refer to the [doc](https://open-policy-agent.github.io/gatekeeper/website/docs/mutation/). We will assume that
Gatekeeper is configured correctly to support mutation. Example 20-6 defines a mutation assignment that matches all Pods except those in the “system” namespace,
and assigns a value of “Always” to imagePullPolicy.

Create the mutation assignment:
```
$ kubectl apply -f imagepullpolicyalways-mutation.yaml
assign.mutations.gatekeeper.sh/demo-image-pull-policy created
```

Now create a Pod. This Pod doesn’t have imagePullPolicy explicitly set, so by default this field is set to “IfNotPresent.” However, we expect Gatekeeper to mutate this fieldto “Always”:

```
$ kubectl apply -f compliant-pod.yaml
pod/kuard created
```

Validate that the imagePullPolicy has been successfully mutated to “Always” by running the following:
```
$ kubectl get pods kuard -o=jsonpath="{.spec.containers[0].imagePullPolicy}"
```

Mutating admission happens before validating admission, so createconstraints that validate the mutations you expect to apply to the specific resource.

### Data Replication

Gatekeeper can be configured to cache specific resources into Open Policy Agent to allow comparisons across resources. The resource in [Example](./config-sync.yaml) configures Gatekeeper to cache Namespace and Pod resources.

### Policy Library

The Gatekeeper project has a great [policy library](https://github.com/open-policy-agent/gatekeeper-library). It contains a general library with the most common policies as well as pod-security-policy library that models the capabilities of the PodSecurityPolicy API as Gatekeeper policy.