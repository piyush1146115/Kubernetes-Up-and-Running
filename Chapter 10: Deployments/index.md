# Deployments

The Deployment object exists to manage the release of new versions. Deployments represent deployed applications in a way that transcends any particular version. Additionally, Deployments enable you to easily move from one version of your code to the next.

### Your First Deployment

Like all objects in Kubernetes, a Deployment can be represented as a declarative YAML object that provides the details about what you want to run. In the following
case, the Deployment is requesting a single instance of the kuard application:

[kuard-deployments](./kuard-deployment.yaml)

Just as we learned that ReplicaSets manage Pods, Deployments manage ReplicaSets. As with all relationships in Kubernetes, this relationship is defined by labels and a label selector. You can see the label selector by looking at the Deployment object:

```
$ kubectl get deployments kuard \
-o jsonpath --template {.spec.selector.matchLabels}
```

You can use this in a label selector query across ReplicaSets to find that specific ReplicaSet the Deployment is managing:

```
$ kubectl get replicasets --selector=run=kuard
```

We can resize the Deployment using the imperative scale command:
```
$ kubectl scale deployments kuard --replicas=2
```

Now if we list that ReplicaSet again, we should see that ReplicaSet has been scaled also:

```
$ kubectl get replicasets --selector=run=kuard
```
Scaling the Deployment has also scaled the ReplicaSet it controls.

Now let’s try the opposite, scaling the ReplicaSet:
```
$ kubectl scale replicasets kuard-1128242161 --replicas=1
```

Now, get that ReplicaSet again:
```
$ kubectl get replicasets --selector=run=kuard
```

That’s odd. Despite scaling the ReplicaSet to one replica, it still has two replicas as its desired state. What’s going on?

Remember, Kubernetes is an online, self-healing system. The top-level Deployment object is managing this ReplicaSet. When you adjust the number of replicas to one, it no longer matches the desired state of the Deployment,which has replicas set to 2. The Deployment controller notices this and takes action to ensure the observed state
matches the desired state, in this case readjusting the number of replicas back to two. If you ever want to manage that ReplicaSet directly, you need to delete the Deploy‐
ment. (Remember to set --cascade to false, or else it will delete the ReplicaSet and Pods as well!)

### Managing Deployments

As with all Kubernetes objects, you can get detailed information about your Deployment via the `kubectl describe` command. This command provides an overview
of the Deployment configuration, which includes interesting fields like the Selector, Replicas, and Events:

```
$ kubectl describe deployments kuard
```

In the output of describe, there is a great deal of important information. Two of the most important pieces of information in the output are `OldReplicaSets` and `New ReplicaSet`. These fields point to the ReplicaSet objects this Deployment is currently managing. If a Deployment is in the middle of a rollout, both fields will be set to a
value. If a rollout is complete, OldReplicaSets will be set to <none>.

In addition to the describe command, there is also the `kubectl rollout` command for Deployments. If you have a current Deployment in progress, you can use `kubectl rollout status` to obtain the current status of that
rollout.

## Updating Deployments

The two most common operations on a Deployment are scaling and application updates.

### Scaling a Deployment

The best practice is to manage your Deployments declaratively via the YAML files, then use those files to update your Deployment. To scale up a Deployment, you would edit your YAML file to increase the number of replicas:

```
...
spec:
replicas: 3
...
```

### Updating a Container Image

The other common use case for updating a Deployment is to roll out a new version of the software running in one or more containers. To do this, you should likewise edit
the Deployment YAML file, though in this case you are updating the container image, rather than the number of replicas:

```
...
containers:
- image: gcr.io/kuar-demo/kuard-amd64:green
  imagePullPolicy: Always
...
```

Annotate the template for the Deployment to record some information about the update:

...
spec:
...
template:
    metadata:
        annotations:
            kubernetes.io/change-cause: "Update togreenkuard"
...

Also, do not update the change-
cause annotation when doing simple scaling operations. A modification of change-cause is a significant change to the template and will trigger a new rollout.

After you update the Deployment, it will trigger a rollout, which you can then monitor via the kubectl rollout command:
```
$ kubectl rollout status deployments kuard
```

If you are in the middle of a rollout and you want to temporarily pause it (e.g., if yo start seeing weird behavior in your system that you want to investigate), you can use the `pause` command:

```
$ kubectl rollout pause deployments kuard
```

If, after investigation, you believe the rollout can safely proceed, you can use the `resume` command to start up where you left off:
```
$ kubectl rollout resume deployments kuard
```

### Rollout History

Kubernetes Deployments maintain a history of rollouts, which can be useful both for understanding the previous state of the Deployment and for rolling back to a specific
version.
You can see the Deployment history by running:
```
$ kubectl rollout history deployment kuard

The revision history is given in oldest to newest order. A unique revision number is incremented for each new rollout.

If you are interested in more details about a particular revision, you can add the `--revision` flag to view details about that specific revision:

```
$ kubectl rollout history deployment kuard --revision=2
``

Let’s say there is an issue with the latest release and you want to roll back while you investigate. You can simply undo the last rollout:
```
$ kubectl rollout undo deployments kuard
```

Let’s look at the Deployment history again:

```
$ kubectl rollout history deployment kuard
deployment.apps/kuard
REVISION CHANGE-CAUSE
1         <none>
3         Update to blue kuard
4         Update to green kuard
```
Revision 2 is missing! It turns out that when you roll back to a previous revision, the Deployment simply reuses the template and renumbers it so that it is the latest
revision. What was revision 2 before is now revision 4.

Additionally, you can roll back to a specific revision in the history using the `--to-revision` flag:

```
$ kubectl rollout undo deployments kuard --to-revision=3
```

Again, the undo took revision 3, applied it, and renumbered it as revision 5. 

By default, the last 10 revisions of a Deployment are kept attached to the Deployment object itself. It is recommended that if you have Deployments that you expect to keep around for a long time, you set a maximum history size for the Deployment revision history. To accomplish this, use the `revisionHistoryLimit` property in the Deployment specification:

```
...
spec:
# We do daily rollouts, limit the revision history to two weeks of
# releases as we don't expect to roll back beyond that.
revisionHistoryLimit: 14
...
```

## Deployment Strategies

Kubernetes deployment supports two different rollout strategies, `Recreate` and `RollingUpdate`

### Recreate Strategy

The Recreate strategy is the simpler of the two. It simply updates the ReplicaSet it manages to use the new image and terminates all of the Pods associated with the Deployment. The ReplicaSet notices that it no longer has any replicas and re-creates all Pods using the new image. Once the Pods are re-created, they are running the new version.

While this strategy is fast and simple, it will result in workload downtime. Because of this, the Recreate strategy should be used only for test Deployments where a service downtime is acceptable.

### RollingUpdate Strategy

The RollingUpdate strategy is the generally preferable strategy for any user-facing service. While it is slower than Recreate, it is also significantly more sophisticated
and robust. Using RollingUpdate, you can roll out a new version of your service while it is still receiving user traffic, without any downtime.

As you might infer from the name, the RollingUpdate strategy works by updating a few Pods at a time, moving incrementally until all of the Pods are running the new
version of your software.

#### Managing multiple versions of your service

Importantly, this means that for a while, both the new and the old version of your service will be receiving requests and serving traffic. This has important implications
for how you build your software. Namely, it is critically important that each version of your software, and each of its clients, is capable of talking interchangeably with
both a slightly older and a slightly newer version of your software.

Your service needs to be backward compatible to ensure zero downtime and to function correctly. This sort of backward compatibility is critical to decoupling your service from systems that depend on your service. If you don’t formalize your APIs and decouple yourself, you are forced to carefully manage your rollouts with all of the other systems that call into your service. This kind of tight coupling makes it extremely hard to produce the necessary agility to be able to push out new software every week, let alone every hour or every day.

#### Configuring a rolling update

RollingUpdate is a fairly generic strategy; it can be used to update a variety of applications in a variety of settings. Consequently, the rolling update itself is quite
configurable; you can tune its behavior to suit your particular needs. There are two parameters you can use to tune the rolling update behavior: `maxUnavailable` and
`maxSurge`.

The `maxUnavailable` parameter sets the maximum number of Pods that can be unavailable during a rolling update. It can either be set to an absolute number (e.g., 3,
meaning a maximum of three Pods can be unavailable) or to a percentage (e.g., 20%, meaning a maximum of 20% of the desired number of replicas can be unavailable). Generally speaking, using a percentage is a good approach for most services, since the value is correctly applied regardless of the desired number of replicas in the Deploy‐
ment. However, there are times when you may want to use an absolute number (e.g., limiting the maximum unavailable Pods to one).

However, there are situations where you don’t want to fall below 100% capacity,but you are willing to temporarily use additional resources to perform a rollout. In these situations, you can set the maxUnavailable parameter to 0 and instead control the rollout using the maxSurge parameter. Like maxUnavailable, maxSurge can be specified either as a specific number or a percentage.

### Slowing Rollouts to Ensure Service Health

Staged rollouts are meant to ensure that the rollout results in a healthy, stable service running the new software version. To do this, the Deployment controller always waits until a Pod reports that it is ready before moving on to update the next Pod. 

The Deployment controller examines the Pod’s status as determined by its readiness checks. f you want to use Deployments to reliably roll out your software, you have to specify readiness health checks for the containers in your Pod. Without these checks, the Deployment controller is running without knowing the Pod’s status.

Sometimes, however, simply noticing that a Pod has become ready doesn’t give you sufficient confidence that the Pod is actually behaving correctly. Some error conditions don’t occur immediately. For example, you could have a serious memory leak that takes a few minutes to show up, or you could have a bug that is only triggered by 1% of all requests. In most real-world scenarios, you want to wait a period of time to have high confidence that the new version is operating correctly before you move on to updating the next Pod.

```
...
spec:
minReadySeconds: 60
...
```

Setting minReadySeconds to 60 indicates that the Deployment must wait for 60 seconds after seeing a Pod become healthy before moving on to updating the next
Pod.

In addition to waiting for a Pod to become healthy, you also want to set a timeout that limits how long the system will wait. Suppose, for example, the new version of your
service has a bug and immediately deadlocks. It will never become ready, and in the absence of a timeout, the Deployment controller will stall your rollout forever.

In order to set the timeout period, you will use the Deployment parameter progres `DeadlineSeconds`:

```
...
spec:
progressDeadlineSeconds: 600
...
```
This example sets the progress deadline to 10 minutes. If any particular stage in the rollout fails to progress in 10 minutes, then the Deployment is marked as failed, and
all attempts to move the Deployment forward are halted. It is important to note that this timeout is given in terms of Deployment progress, not the overall length of a Deployment. In this context, progress is defined as any time the Deployment creates or deletes a Pod. When that happens, the timeout clock is reset to zero.

## Deleting a Deployment

If you ever want to delete a Deployment, you can do it with the imperative command:

```
$ kubectl delete deployments kuard
```
By default, deleting a Deployment deletes the entire service. The means it will delete not just the Deployment, but also any ReplicaSets it manages, as well as any Pods the ReplicaSets manage. As with ReplicaSets, if this is not the desire behavior, you can use the `--cascade=false` flag to delete only the Deployment object.

