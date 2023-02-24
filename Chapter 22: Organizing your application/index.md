# Organizing Your Application

Obviously, reliability and agility are the general goals of developing a cloud native application in Kubernetes. The following sections describe three principles that can guide you in designing a structure that best suits these goals. The principles are:

- Treat filesystems as the source of truth
- Conduct code review to ensure the quality of changes
- Use feature flags to stage rollouts and rollbacks

### Filesystems as the Source of Truth

Rather than viewing the state of the cluster—the data in etcd —as the source of truth, it is optimal to view the filesystem of YAML objects as the source of truth for your application. The API objects deployed into your Kubernetes cluster(s) are then a reflection of the truth stored in the filesystem. It is absolutely a first principle that all applications deployed to Kubernetes should first be described in files stored in a filesystem. The
actual API objects are then just a projection of this filesystem into a particular cluster.

### The Role of Code Review

Most service outages are self-inflicted via unexpected consequences, typos, or other simple mistakes. Ensuring that at least two people look at any configuration change significantly decreases the probability of such errors.

### Feature Gates

The idea is that when some new feature is developed, that development takes place entirely behind a feature flag or gate. This gate looks something like:

```
if (featureFlags.myFlag) {
// Feature implementation goes here
}
```

Working behind a feature flag also means that enabling a feature simply involves making a configuration change to activate the flag. This makes it very clear what
changed in the production environment, and very simple to roll back the feature activation if it causes problems.

Using feature flags thus both simplifies debugging and ensures that disabling a feature doesn’t require a binary rollback to an older version of the code that would remove all of the bug fixes and other improvements made by the newer version.

## Managing Your Application in Source Control

Obviously, filesystems contain hierarchical directories, and a source-control system adds concepts like tags and branches, so this section describes
how to put these together to represent and manage your application.

### Filesystem Layout

Thus, for an application with a frontend that uses two services, the filesystem might look like this:

```
frontend/
service-1/
service-2/
```

Within each of these directories, the configurations for each application are stored. These are the YAML files that directly represent the current state of the cluster. It’s generally useful to include both the service name and the object type within the same file. Thus, extending our previous example, the filesystem might look like:

```
frontend/
    frontend-deployment.yaml
    frontend-service.yaml
    frontend-ingress.yaml
service-1/
    service-1-deployment.yaml
    service-1-service.yaml
    service-1-configmap.yaml
...
```

## Managing Periodic Versions

It’s handy to be able to simultaneously store and maintain multiple revisions of your configuration. There are two different approaches that you can use
with the file and version control systems we’ve outlined here:
- Use tags, branches, and source-control features
- Clone the configuration within the filesystem and use directories for different revisions.

### Versioning with branches and tags

When you use branches and tags to manage configuration revisions, the directory structure does not change from the example in the previous section. When you are
ready for a release, you place a source-control tag (such as git tag v1.0 ) in the configuration source-control system. The tag represents the configuration used for that version, and the HEAD of source control continues to iterate forward.