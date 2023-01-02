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