# Deploying a Kubernetes Cluster

At this point, there are cloud-based Kubernetes services in most public clouds that make it easy to create a cluster with a few command-line instructions.

## Installing Kubernetes on a Public Cloud Provider

This chapter covers installing Kubernetes on the three major cloud providers: the Google Cloud Platform, Microsoft Azure, and Amazon Web Services.

### Installing Kubernetes with Google Kubernetes Engine

Once you have gcloud installed, set a default zone:

```
$ gcloud config set compute/zone us-west1-a
```

Then you can create a cluster:

```
$ gcloud container clusters create kuar-cluster --num-nodes=3`
```

This will take a few minutes. When the cluster is ready, you can get credentials for thecluster using:

```
$ gcloud container clusters get-credentials kuar-cluster`
```

### Installing Kubernetes with Azure Kubernetes Service

When you have the shell up and working, you can run:
```
$ az group create --name=kuar --location=westus
```

Once the resource group is created, you can create a cluster using:
```
$ az aks create --resource-group=kuar --name=kuar-cluster
```

This will take a few minutes. Once the cluster is created, you can get credentials for the cluster with:
```
$ az aks get-credentials --resource-group=kuar --name=kuar-cluster
```

If you don’t already have the kubectl tool installed, you can install it using:
```
$ az aks install-cli
```

### Installing Kubernetes on Amazon Web Services

Amazon offers a managed Kubernetes service called Elastic Kubernetes Service(EKS). The easiest way to create an EKS cluster is via the open source eksctl
command-line tool.

Once you have eksctl installed and in your path, you can run the following command to create a cluster:

```
$ eksctl create cluster
```

For more details on installation options (such as node size and more), view the help using this command:
```
$ eksctl create cluster --help
```

## Installing Kubernetes Locally Using minikube

You can create a local cluster using:

```
$ minikube start
```

This will create a local VM, provision Kubernetes, and create a local kubectl configuration that points to that cluster. As mentioned previously, this cluster only has a single node, so while it is useful, it has some differences with most production deployments of Kubernetes.

When you are done with your cluster, you can stop the VM with:
```
$ minikube stop
```

## Running Kubernetes in Docker

The [kind project](https://kind.sigs.k8s.io) provides a great experience for launching and managing test clusters in Docker. (kind stands for Kubernetes IN Docker.)

Once you get it installed, creating a cluster is as easy as:
```
$ kind create cluster --wait 5m
$ export KUBECONFIG="$(kind get kubeconfig-path)"
$ kubectl cluster-info
$ kind delete cluster
```

## The Kubernetes Client

The official Kubernetes client is `kubectl` : a command-line tool for interacting with the Kubernetes API.

### Checking Cluster Status

You can check the version of the cluster that you are running:
```
$ kubectl version
```

To verify that your cluster is generally healthy:

```
$ kubectl get componentstatuses
```

### Listing Kubernetes Nodes

You can list all of the nodes in your cluster:

```
$ kubectl get nodes
```

In Kubernetes, nodes are separated into control-plane nodes that contain containers like the API server, scheduler, etc., which manage the cluster, and worker nodes where your containers will run. Kubernetes won’t generally schedule work onto control-plane nodes to ensure that user workloads don’t harm the overall operation of the cluster.

You can use the kubectl describe command to get more information about a specific node, such as kind-control-plane :
```
$ kubectl describe nodes kind-control-plane
```

## Cluster Components

One of the interesting aspects of Kubernetes is that many of the components that make up the Kubernetes cluster are actually deployed using Kubernetes itself.

### Kubernetes Proxy

The Kubernetes proxy is responsible for routing network traffic to load-balanced services in the Kubernetes cluster. Kubernetes has an API object named DaemonSet, which is used in many clusters to accomplish this.

```
$ kubectl get daemonSets --namespace=kube-system kube-proxy
```

### Kubernetes DNS

Kubernetes also runs a DNS server, which provides naming and discovery for the services that are defined in the cluster. This DNS server also runs as a replicated
service on the cluster. Depending on the size of your cluster, you may see one or more DNS servers running in your cluster. The DNS service is run as a Kubernetes
deployment, which manages these replicas.

```
$ kubectl get deployments --namespace=kube-system coredns
```

There is also a Kubernetes service that performs load balancing for the DNS server:

```
$ kubectl get services --namespace=kube-system kube-dns
```

### Kubernetes UI

There is a community supported GUI for Kubernetes - [documentation](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)