# Labels and Annotations

Labels are key/value pairs that can be attached to Kubernetes objects such as Pods and ReplicaSets. Labels provide the foundation for grouping objects.

Annotations, on the other hand, provide a storage mechanism that resembles labels: key/value pairs designed to hold nonidentifying information that tools and libraries can leverage. Unlike labels, annotations are not meant for querying, filtering, or otherwise differentiating Pods from each other.

## Labels

Labels provide identifying metadata for objects. These are fundamental qualities of the object that will be used for grouping, viewing, and operating.

Labels have simple syntax. They are key/value pairs, where both the key and value are represented by strings. Label keys can be broken down into two parts: an optional prefix and a name, separated by a slash. The key name is required and have a maximum length of 63 characters. Names must also start and end with an alphanumeric character and permit the use of dashes ( - ), underscores ( _ ), and dots ( . ) between characters. Label values are strings with a maximum length of 63 characters.

### Applying Labels

Here we create a few deployments (a way to create an array of Pods) with some interesting labels. We’ll take two apps (called alpaca and bandicoot ) and have two environments and two versions for each.

```bash
$ kubectl run alpaca-prod \
--image=gcr.io/kuar-demo/kuard-amd64:blue \
--replicas=2 \
--labels="ver=1,app=alpaca,env=prod"


$ kubectl run alpaca-test \
--image=gcr.io/kuar-demo/kuard-amd64:green \
--replicas=1 \
--labels="ver=2,app=alpaca,env=test"

$ kubectl run bandicoot-prod \
--image=gcr.io/kuar-demo/kuard-amd64:green \
--replicas=2 \
--labels="ver=2,app=bandicoot,env=prod"

$ kubectl run bandicoot-staging \
--image=gcr.io/kuar-demo/kuard-amd64:green \
--replicas=1 \
--labels="ver=2,app=bandicoot,env=staging"
```

To see the Pods with Labels:
```bash
$ kubectl get deployments --show-labels
```


### Modifying Labels

You can also apply or update labels on objects after you create them:

```bash
$ kubectl label deployments alpaca-test "canary=true"
```

You can remove a label by applying a dash-suffix:
```bash
$ kubectl label deployments alpaca-test "canary-"
```

### Label Selectors

Label selectors are used to filter Kubernetes objects based on a set of labels. Selectors use a simple syntax for Boolean expressions. They are used both by end users (via tools like kubectl ) and by different types of objects (such as how a ReplicaSet relates to its Pods).

If we want to list only Pods that have the ver label set to 2 , we could use the `--selector flag`:
```
$ kubectl get pods --selector="ver=2"
```

If we specify two selectors separated by a comma, only the objects that satisfy both will be returned. This is a logical AND operation:
```
$ kubectl get pods --selector="app=bandicoot,ver=2"
```

We can also ask if a label is one of a set of values. Here we ask for all Pods where the app label is set to alpaca or bandicoot:
```
$ kubectl get pods --selector="app in (alpaca,bandicoot)"
```

Finally, we can ask if a label is set at all. Here we are asking for all of the deployments with the canary label set to anything:

```
$ kubectl get deployments --selector="canary"
```

There are also `negative` versions of each of these.

### Label Selectors in API Objects

A Kubernetes object uses a label selector to refer to a set of other Kubernetes objects. Instead of a simple string as described in the previous section, we use a parsed structure. A selector of app=alpaca,ver in (1, 2) would be converted to this:

```yaml
selector:
  matchLabels:
    app: alpaca
  matchExpressions:
    - {key: ver, operator: In, values: [1, 2]}
```

### Labels in the Kubernetes Architecture\

Kubernetes is a purposefully decoupled system. There is no hierarchy and all components operate independently. However, in many cases, objects need to relate to one another, and these relationships are defined by labels and label selectors.

For example, ReplicaSets, which create and maintain multiple replicas of a Pod, find the Pods that they are managing via a selector. Likewise, a service load balancer finds the Pods to which it should bring traffic via a selector query.

## Annotations

Annotations provide a place to store additional metadata for Kubernetes objects where the sole purpose of the metadata is assisting tools and libraries. They are a way for other programs driving Kubernetes via an API to store some opaque data with an object.

While labels are used to identify and group objects, annotations are used to provide extra information about where an object came from, how to use it, or policy around that object. There is overlap, and it is a matter of taste as to when to use an annotation or a label. When in doubt, add information to an object as an annotation and promote it to a label if you find yourself wanting to use it in a selector.

Annotations are defined in the common metadata section in every Kubernetes object:
```yaml
metadata:
  annotations:
    example.com/icon-url: "https://example.com/icon.png"
```