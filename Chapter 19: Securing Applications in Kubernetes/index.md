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