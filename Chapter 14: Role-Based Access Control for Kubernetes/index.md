# Role-Based Access Control for Kubernetes

Role-based access control provides a mechanism for restricting both access to and actions on Kubernetes APIs to ensure that only authorized users have access. RBAC
is a critical component to both harden access to the Kubernetes cluster where you are deploying your application and (possibly more importantly) prevent unexpected accidents where one person in the wrong namespace mistakenly takes down production when they think they are destroying their test cluster.

Every request to Kubernetes is first authenticated. Authentication provides the identity of the caller issuing the request. It could be as simple as saying that the request is unauthenticated, or it could integrate deeply with a pluggable authentication provider (e.g., Azure Active Directory) to establish an identity within that third-party system.

Once users have been authenticated, the authorization phase determines whether they are authorized to perform the request. Authorization is a combination of the
identity of the user, the resource (effectively the HTTP path), and the verb or action the user is attempting to perform. If the particular user is authorized to perform that action on that resource, then the request is allowed to proceed. Otherwise, an HTTP
403 error is returned.

## Role-Based Access Control

To properly manage access in Kubernetes, it’s critical to understand how identity, roles, and role bindings interact to control who can do what with which resources.

### Identity in Kubernetes

Every request to Kubernetes is associated with some identity. Even a request with no identity is associated with the `system:unauthenticated` group. Kubernetes makes a distinction between user identities and service account identities. Service accounts are created and managed by Kubernetes itself and are generally associated with components running inside the cluster. User accounts are all other accounts associated with actual users of the cluster, and often include automation like continuous delivery services that run outside the cluster.

Kubernetes uses a generic interface for authentication providers. Each of the providers supplies a username and, optionally, the set of groups to which the user belongs.

Kubernetes supports a number of authentication providers, including:
-  HTTP Basic Authentication (largely deprecated)
-  x509 client certificates
-  Static token files on the host
-  Cloud authentication providers, such as Azure Active Directory and AWS Identity and Access Management (IAM)
-  Authentication webhooks

### Roles and Role Bindings in Kubernetes

Identity is just the beginning of authorization in Kubernetes. Once Kubernetes knows the identity of the request, it needs to determine if the request is authorized for that user. To achieve this, it uses roles and role bindings.

In Kubernetes, two pairs of related resources represent roles and role bindings. One pair is scoped to a namespace (Role and RoleBinding), while the other pair is scoped to the cluster (ClusterRole and ClusterRoleBinding).

Role resources are namespaced and represent capabilities within that single namespace. You cannot use namespaced roles for nonnamespaced resources (e.g., CustomResourceDefinitions), and binding a Role‐
Binding to a role only provides authorization within the Kubernetes namespace that contains both the Role and the RoleBinding.

Sometimes you need to create a role that applies to the entire cluster, or you want to limit access to cluster-level resources. To achieve this, you use the ClusterRole and ClusterRoleBinding resources. They are largely identical to their namespaced peers, but are cluster-scoped.

#### Verbs for Kubernetes roles

Roles are defined in terms of both a resource (e.g., Pods) and a verb that describes
an action that can be performed on that resource.

#### Using built-in roles

Kubernetes has a large number of built-in cluster roles for well-known system identities (e.g., a
scheduler) that require a known set of capabilities. You can view these by running:

```
$ kubectl get clusterroles
```

Most clusters already have numerous ClusterRole bindings set up, and you can view
these bindings with `kubectl get clusterrolebindings`.

#### Auto-reconciliation of built-in roles

When the Kubernetes API server starts up, it automatically installs a number of default ClusterRoles that are defined in the code of the API server itself. This means that if you modify any built-in cluster role, those modifications are transient.
Whenever the API server is restarted (e.g., for an upgrade), your changes will be overwritten. To prevent this from happening, before you make any other modifications, you need to add the rbac.authorization.kubernetes.io/autoupdate annotation with a
value of false to the built-in ClusterRole resource. If this annotation is set to false, the API server will not overwrite the modified ClusterRole resource.

By default, the Kubernetes API server installs a cluster role that allows system:unauthenticated users access to the API server’s API discovery endpoint. For any cluster exposed to a hostile environment (e.g., the public internet) this is a bad idea, and there has
been at least one serious security vulnerability via this exposure. If you are running a Kubernetes service on the public internet or an other hostile environment, you should ensure that the `--anonymous-auth=false` flag is set on your API server.

## Techniques for Managing RBAC

Fortunately, there are several tools and techniques that make managing RBAC easier.

### Testing Authorization with can-i

The first useful tool is the `auth can-i` command for kubectl. This tool is used for testing whether a specific user can perform a specific action. You can use `can-i` to validate configuration settings as you configure your cluster, or you can ask users to use the tool to validate their access when filing errors or bug reports.

In its simplest usage, the can-i command takes a verb and a resource. For example, this command will indicate if the current kubectl user is authorized to create Pods:

```
$ kubectl auth can-i create pods
```

You can also test subresources like logs or port-forwarding with the --subresource
command-line flag:

```
$ kubectl auth can-i get pods --subresource=logs
```

### Managing RBAC in Source Control

The kubectl command-line tool provides a reconcile command that operates somewhat like kubectl apply and will reconcile a set of roles and role bindings
with the current state of the cluster. You can run:
```
$ kubectl auth reconcile -f some-rbac-config.yaml
```

## Advanced Topics

But when managing a large number of users or roles, there are additional advanced capabilities you can use to manage RBAC at scale.

### Aggregating ClusterRoles

Kubernetes RBAC supports the usage of an aggregation rule to combine multiple roles in a new role. This new role combines all of the capabilities of all of the aggregate roles, and any changes to any of the constituent subroles will automatically be propogated back into the aggregate role.

In this particular case, the `aggregationRule` field in the ClusterRole resource contains a `clusterRoleSelector` field, which in turn is a label selector. All ClusterRole resources that match this selector are dynamically aggregated into the rules array in the aggregate ClusterRole resource.

A best practice for managing ClusterRole resources is to create a number of fine-grained cluster roles and then aggregate them to form higher-level or broader cluster roles. This is how the built-in cluster roles are defined. For example, you can see that
the built-in edit role looks like this:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: edit
...
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      rbac.authorization.k8s.io/aggregate-to-edit: "true"
...
```

This means that the edit role is defined to be the aggregate of all ClusterRole objects that have a label of `rbac.authorization.k8s.io/aggregate-to-edit` set to true.

### Using Groups for Bindings

When you bind a group to a Role or ClusterRole, anyone who is a member of that group gains access to the resources and verbs defined by that role. Thus, to enable any individual to gain access to the group’s role, that individual needs to be added to the group.

Many group systems enable “just in time” (JIT) access, such that people are only temporarily added to a group in response to an event (say, a page in the middle of the
night) rather than having standing access. This means that you can both audit who had access at any particular time and ensure that, in general, even a compromised
identity can’t have access to your production infrastructure.

Finally, in many cases, these same groups are used to manage access to other resources, from facilities to documents and machine logins. Thus, using the same
groups for access control to Kubernetes dramatically simplifies management.

To bind a group to a ClusterRole, use a Group kind for the subject in the binding:
```
...
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: my-great-groups-name
...
```