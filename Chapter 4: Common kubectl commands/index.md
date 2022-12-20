# Common kubectl commands

The kubectl command-line utility is a powerful tool.

## Namespaces

By default, the kubectl command-line tool interacts with the `default` namespace. If you want to use a different namespace, you can pass kubectl the `--namespace` flag.

## Contexts

If you want to change the default namespace more permanently, you can use a context. This gets recorded in a kubectl configuration file, usually located at
`$HOME/.kube/config`. This configuration file also stores how to both find and authen‐ticate to your cluster. For example, you can create a context with a different default
namespace for your kubectl commands using:
```
$ kubectl config set-context my-context --namespace=mystuff
```

To use this newly created context, you can run:
```
$ kubectl config use-context my-context
```

## Viewing Kubernetes API Objects

Everything contained in Kubernetes is represented by a RESTful resource. Each Kubernetes
object exists at a unique HTTP path; for example, https://your-k8s.com/api/v1/name‐spaces/default/pods/my-pod leads to the representation of a Pod in the default name‐
space named my-pod . The kubectl command makes HTTP requests to these URLs to access the Kubernetes objects that reside at these paths.

By default, kubectl uses a human-readable printer for viewing the responses from the API server. One way to get slightly more information is to add the -o wide flag, which gives more details, on a longer line.

Another common task is extracting specific fields from the object. kubectl uses the
JSONPath query language to select fields in the returned object. As an example, this command will extract and print the IP address of the specified Pod:
```
$ kubectl get pods my-pod -o jsonpath --template={.status.podIP}
```

If you are interested in more detailed information about a particular object, use the describe command:
```
$ kubectl describe <resource-name> <obj-name>
```

If you would like to see a list of supported fields for each supported type of Kubernetes object, you can use the explain command:
```
$ kubectl explain pods
```

To continuously monitor the state of a particular resource, use the `--watch` flag with kubectl command.


## Creating, Updating, and Destroying Kubernetes Objects

Objects in the Kubernetes API are represented as JSON or YAML files. These files are either returned by the server in response to a query or posted to the server as part of an API request. You can use these YAML or JSON files to create, update, or delete objects on the Kubernetes server.

Let’s assume that you have a simple object stored in obj.yaml. You can use kubectl to create this object in Kubernetes by running:
```
$ kubectl apply -f obj.yaml
```

Similarly, after you make changes to the object, you can use the apply command again to update the object:
```
$ kubectl apply -f obj.yaml
```

The apply tool will only modify objects that are different from the current objects in the cluster. If the objects you are creating already exist in the cluster, it will simply exit successfully without making any changes.

You can use the `--dry-run` flag to print the objects to the terminal without actually sending them to the server.

If you feel like making interactive edits instead of editing a local file, you can instead use the edit command, which will download
the latest object state and then launch an editor that contains the definition:
```
$ kubectl edit <resource-name> <obj-name>
```
After you save the file, it will be automatically uploaded back to the Kubernetes cluster.

The apply command also records the history of previous configurations in an annotation within the object. You can manipulate these records with the `edit-last-
applied` , `set-last-applied` , and `view-last-applied` commands. For example:
```
$ kubectl apply -f myobj.yaml view-last-applied
```

When you want to delete an object, you can simply run:
```
$ kubectl delete -f obj.yaml
```

Likewise, you can delete an object using the resource type and name:
```
$ kubectl delete <resource-name> <obj-name>
```

## Labeling and Annotating Objects

Labels and annotations are tags for your objects. You can update the labels and annotations on any Kubernetes object using the label and annotate commands. For example, to add the color=red label to a Pod named bar , you can run:
```
$ kubectl label pods bar color=red
```

The syntax for annotations is identical.

By default, label and annotate will not let you overwrite an existing label. To do this, you need to add the `--overwrite` flag.
If you want to remove a label, you can use the `<label-name>-` syntax:
```
$ kubectl label pods bar color-
```