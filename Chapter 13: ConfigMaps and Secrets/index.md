# ConfigMaps and Secrets

ConfigMaps are used to provide configuration information for workloads. This can be either fine-grained information like a string or a composite value in the form of a file. Secrets are similar to ConfigMaps but focus on making sensitive information available to the workload.
They can be used for things like credentials or TLS certificates.

## ConfigMaps

One way to think of a ConfigMap is as a Kubernetes object that defines a small filesystem. Another way is as a set of variables that can be used when defining the environment or command line for your containers.

### Creating ConfigMaps

First, suppose we have a file on disk (called my-config.txt) that we want to make available to the Pod.

Next, let’s create a ConfigMap with that file. We’ll also add a couple of simple key/value pairs here. These are referred to as literal values on the command line:

```
$ kubectl create configmap my-config \
--from-file=my-config.txt \
--from-literal=extra-param=extra-value \
--from-literal=another-param=another-value
```

The equivalent YAML for the ConfigMap object we just created is as follows:
```
$ kubectl get configmaps my-config -o yaml

apiVersion: v1
data:
  another-param: another-value
  extra-param: extra-value
  my-config.txt: |-
    # This is a sample config file that I might use to configure an application
    parameter1 = value1
    parameter2 = value2
kind: ConfigMap
metadata:
  creationTimestamp: "2023-01-27T14:43:16Z"
  name: my-config
  namespace: default
  resourceVersion: "507356"
  uid: 975e9820-5414-402c-9800-2f0186d0f8d8
```

As you can see, the ConfigMap is just some key/value pairs stored in an object.

### Using a ConfigMap

There are three main ways to use a ConfigMap:

- **Filesystem**: You can mount a ConfigMap into a Pod. A file is created for each entry based on the key name. The contents of that file are set to the value.
- **Environment variable**: A ConfigMap can be used to dynamically set the value of an environment variable.
- **Command-line argument**: Kubernetes supports dynamically creating the command line for a container based on ConfigMap values.

Let’s create a manifest for kuard that pulls all of these together, as shown in
[Example](./kuard-config.yaml).

For the filesystem method, we create a new volume inside the Pod and give it the name `config-volume`. We then define this volume to be a ConfigMap volume and point at the ConfigMap to mount. We have to specify where this gets mounted into the kuard container with a volumeMount. In this case, we are mounting it at `/config`. Environment variables are specified with a special valueFrom member. This references the ConfigMap and the data key to use within that ConfigMap. Command-line arguments build on environment variables. Kubernetes will perform the correct substitution with a special $(<env-var-name>) syntax.

## Secrets

While ConfigMaps are great for most configuration data, there is certain data that is extra sensitive. This includes passwords, security tokens, or other types of private keys. Collectively, we call this type of data “Secrets.”

Secrets enable container images to be created without bundling sensitive data. This allows containers to remain portable across environments. Secrets are exposed to
Pods via explicit declaration in Pod manifests and the Kubernetes API.

### Creating Secrets

Secrets are created using the Kubernetes API or the kubectl command-line tool. Secrets hold one or more data elements as a collection of key/value pairs.

With the kuard.crt and kuard.key files stored locally, we are ready to create a Secret. Create a Secret named kuard-tls using the `create secret` command:

```
$ kubectl create secret generic kuard-tls \
--from-file=kuard.crt \
--from-file=kuard.key
```

The kuard-tls Secret has been created with two data elements. Run the following command to get details:

```
$ kubectl describe secrets kuard-tls
```

### Consuming Secrets

Secrets can be consumed using the Kubernetes REST API by applications that know how to call that API directly. However, our goal is to keep applications portable. Not
only should they run well in Kubernetes, but they should run, unmodified, on other platforms.

Instead of accessing Secrets through the API server, we can use a Secrets volume. Secret data can be exposed to Pods using the Secrets volume type.

Each data element of a Secret is stored in a separate file under the target mount point specified in the volume mount. The kuard-tls Secret contains two data elements:
`kuard.crt` and `kuard.key`. Mounting the kuard-tls Secrets volume to /tls results in the following files:
/tls/kuard.crt
/tls/kuard.key

The Pod manifest in [Example](./kuard-secret.yaml) demonstrates how to declare a Secrets volume, which exposes the kuard-tls Secret to the kuard container under /tls.

### Private Container Registries

Image pull Secrets leverage the Secrets API to automate the distribution of private registry credentials. Image pull Secrets are stored just like regular Secrets but are
consumed through the `spec.imagePullSecrets` Pod specification field.

Use kubectl create secret docker-registry to create this special kind of Secret:

```
$ kubectl create secret docker-registry my-image-pull-secret \
--docker-username=<username> \
--docker-password=<password> \
--docker-email=<email-address>
```

Enable access to the private repository by referencing the image pull secret in the Pod manifest file, as shown in [Example](./kuard-secret-ips.yaml).

## Managing ConfigMaps and Secrets

ConfigMaps and Secrets are managed through the Kubernetes API. The usual create, delete, get, and describe commands work for manipulating these objects.

### Listing

You can use the `kubectl get secrets` command to list all Secrets in the current namespace. Similarly, you can list all of the ConfigMaps in a namespace:

```
$ kubectl get secrets
$ kubectl get configmaps
```

kubectl describe can be used to get more details on a single object:

```
$ kubectl describe configmap my-config
```

Finally, you can see the raw data (including values in Secrets!) by using a command similar to the following: `kubectl get configmap my-config -o yaml` or `kubectl get secret kuard-tls -o yaml`

### Creating

The easiest way to create a Secret or a ConfigMap is via `kubectl create secret generic` or `kubectl create configmap`. There are a variety of ways to specify the
data items that go into the Secret or ConfigMap. These can be combined in a single command:

- `--from-file=<filename>`
Load from the file with the Secret data key that’s the same as the filename.
- `--from-file=<key>=<filename>`
Load from the file with the Secret data key explicitly specified.
- `--from-file=<directory>`
Load all the files in the specified directory where the filename is an acceptable
key name.
- `--from-literal=<key>=<value>`
Use the specified key/value pair directly.

### Updating

You can update a ConfigMap or Secret and have it reflected in running applications.

#### Update from file

If you have a manifest for your ConfigMap or Secret, you can just edit it directly and replace it with a new version using `kubectl replace -f <filename>`. You can
also use `kubectl apply -f <filename>` if you previously created the resource with
kubectl apply.

#### Re-create and update

If you store the inputs into your ConfigMaps or Secrets as separate files on disk, you can use kubectl to re-create the manifest and then use it to update the object, which will look something like this:

```
$ kubectl create secret generic kuard-tls \
--from-file=kuard.crt --from-file=kuard.key \
--dry-run -o yaml | kubectl replace -f -
```

This command line first creates a new Secret with the same name as our existing Secret. We then pipe that to kubectl
replace and use -f - to tell it to read from stdin. In this way, we can update a Secret from files on disk without having to manually base64-encode data.

#### Edit current version

The final way to update a ConfigMap is to use `kubectl edit` to bring up a version of the ConfigMap in your editor so you can tweak it:

```
$ kubectl edit configmap my-config
```

#### Live updates

Once a ConfigMap or Secret is updated using the API, it’ll be automatically pushed to all volumes that use that ConfigMap or Secret. It may take a few seconds, but
the file listing and contents of the files, as seen by kuard, will be updated with these new values. Using this live update feature, you can update the configuration of
applications without restarting them.

Currently there is no built-in way to signal an application when a new version of a ConfigMap is deployed. It is up to the application (or some helper script) to look for the config files to change and reload them.