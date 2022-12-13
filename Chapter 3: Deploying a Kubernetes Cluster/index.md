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
`$ gcloud container clusters create kuar-cluster --num-nodes=3`
```

This will take a few minutes. When the cluster is ready, you can get credentials for thecluster using:

```
`$ gcloud container clusters get-credentials kuar-cluster`
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

### Installing Kubernetes Locally Using minikube

You can create a local cluster using:

```
$ minikube start
```

This will create a local VM, provision Kubernetes, and create a local kubectl configuration that points to that cluster. As mentioned previously, this cluster only has a single node, so while it is useful, it has some differences with most production deployments of Kubernetes.
