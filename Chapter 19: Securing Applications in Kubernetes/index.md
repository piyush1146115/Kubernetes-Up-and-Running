# Securing Applications in Kubernetes

It’s important to understand the following two concepts when securing Pods in Kubernetes: defense in depth and principle of least privilege. Defense in depth is a
concept where you use multiple layers of security controls across your computing systems that include Kubernetes. The principle of least privilege means giving your workloads access only to resources that are required for them to operate. Both these concepts are not destinations, but constantly applied to the ever-changing computing system landscape.

## Understanding SecurityContext

At the core of securing Pods is SecurityContext, which is an aggregation of all security-focused fields that may be applied at both the Pod and container specification level. Here are some example security controls covered by SecurityContext:
- User permissions and access control (e.g., setting User ID and Group ID)
- Read-only root filesystem
- Allow privilege escalation
- Seccomp, AppArmor, and SELinux profile and label assignments
- Run as privileged or unprivileged

You can see in this [example](./kuard-pod-securitycontext.yaml) that there is a SecurityContext at both the Pod and the container level. Many of the security controls can be applied at both of these levels. In the case that they are applied in both, the container level configuration takes
precedence.

- runAsNonRoot: The Pod or container must run as a nonroot user. The container will fail to start if it is running as a root user.
- runAsUser/runAsGroup: This setting overrides the user and group that the container process is run as. Container images may have this configured as part of the Dockerfile.
- fsgroup: Configures Kubernetes to change the group of all files in a volume when they are mounted into a Pod.
- allowPrivilegeEscalation: Configures whether a process in a container can gain more privileges than its parent.
- privileged: Runs the container as privileged, which elevates the container to the same permissions as the host.
- readOnlyRootFilesystem: Mounts the container root filesystem to read-only. This is a common attack vector and is best practice to enable.

We will introduce the operating system level security controls and then how to configure them via SecurityContext. It’s important to note that many of these controls are host operating system dependent. Here are a list of the core set of operating system controls that are covered by SecurityContext:

- Capabilities: Allow either the addition or removal of groups of privilege that may be required for a workload to operate.
- AppArmor: Controls which files processes can access. AppArmor profiles can be applied to containers via the addition of an annotation of `container.apparmor.security.beta.kubernetes.io/<container_name>: <profile_ref>` to the Pod specification.
- Seccomp: Seccomp (secure computing) profiles allow the creation of syscall filters. These filters allow specific syscalls to be allowed or blocked, which limits the surfacearea of the Linux kernel that is exposed to the processes in the Pods.
- SELinux: Defines access controls for files and processes. SELinux operators use labels that are grouped together to create a security context (not to be mistaken with a Kubernetes SecurityContext), which is used to limit access to a process. By default, Kubernetes allocates a random SELinux context for each container; how‐ever, you may choose to set one via SecurityContext.

### SecurityContext Challenges

The creation and management of AppArmor, seccomp, and SELinux profiles and contexts is not easy and is error prone. The cost of an error is breaking the
ability for an application to perform its function. There are several tools out there that create a way to generate a seccomp profile from a running Pod, which can then be applied using SecurityContext. One such project is the [Security Profiles Operator](https://github.com/kubernetes-sigs/security-profiles-operator), which makes it easy to generate and manage Seccomp profiles.

## Pod Security

One of the main differences between Pod Security and its predecessor is that Pod Security only performs validation and not mutation.

### What Is Pod Security?

Pod Security allows you to declare different security profiles for Pods. These security profiles are known as Pod Security Standards and are applied at the namespace level. Pod Security Standards are a collection of security-sensitive fields in a Pod specification (including, but not limited to, SecurityContext) and their associated values. There are three different standards that range from restricted to permissive. The idea is that you can apply a general security posture to all Pods in a given namespace. The three Pod Security Standards are as follows:

- Baseline: Most common privilege escalation while enabling easier onboarding.
- Restricted: Highly restricted, covering security best practices. May cause workloads to break.
- Privileged: Open and unrestricted.

Each Pod Security Standard defines a list of fields in the Pod specification and their allowed values. Here are some fields that are covered by these standards:
- spec.securityContext
- spec.containers[*].securityContext
- spec.containers[*].ports
- spec.volumes[*].hostPath

Each standard is applied to a namespace using a given mode. There are three modes a policy may be applied to. They are as follows:
- Enforce: Any Pods that violate the policy will be denied
- Warn: Any Pods that violate the policy will be allowed, and a warning message will be displayed to the user
- Audit: Any Pods that violate the policy will generate an audit message in the audit log

### Applying Pod Security Standards

Pod Security Standards are applied to a namespace using labels as follows:
- Required: pod-security.kubernetes.io/<MODE>: <LEVEL>
- Optional: pod-security.kubernetes.io/<MODE>-version: <VERSION> (defaults to latest)

The namespace in [Example](./baseline-ns.yaml) illustrates how you may use multiple modes to enforce at one standard (baseline in this example) and audit and warn at another(restricted).

Pod Security is a great way to manage the security posture of your workloads byapplying policy at the namespace level and allowing Pods to be created only if
they don’t violate the policy. It’s flexible and offers different prebuilt policies from permissive to restricted along with tooling to easily roll out policy changes without the risk of breaking workloads.

### Service Account Management

Service accounts are Kubernetes resources that provide an identity to workloads that run inside Pods. RBAC can be applied to service accounts to control what resources, via the Kubernetes API, the identity has access to. By default, Kubernetes creates a default service account in each namespace, which is automatically set as the serviceaccount for all Pods. This service account contains a token that is automounted in each Pod and is used to access the Kubernetes API. To disable this behavior, you must add `automountServiceAccountToken: false` to the service account configuration.

This [Example](./service-account.yaml) demonstrates how this can be done for the default service account. 

Service accounts are often overlooked when considering Pod security; however, they allow direct access to the Kubernetes API and, without adequate RBAC, could allow an attacker access to Kubernetes.