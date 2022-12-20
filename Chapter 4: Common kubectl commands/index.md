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
JSONPath query language to select fields in the returned object. The complete details of JSONPath are beyond the scope of this chapter, but as an example, this command will extract and print the IP address of the specified Pod:
```
$ kubectl get pods my-pod -o jsonpath --template={.status.podIP}
```