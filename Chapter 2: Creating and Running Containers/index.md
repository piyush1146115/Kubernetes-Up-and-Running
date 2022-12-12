# Creating and Running Containers

Container images bundle a program and its dependencies into a single artifact under a root filesystem. The most popular container image format is the Docker image
format, which has been standardized by the Open Container Initiative to the OCI image format. Kubernetes supports both Docker- and OCI-compatible images via
Docker and other runtimes.

## Container images

A container image is a binary package that encapsulates all of the files necessary to run a program inside of an OS container. The most popular and widespread container image format is the Docker image format, which was developed by the Docker open source project for packaging, distributing, and running containers using the docker command. Docker image format continues to be the de facto standard and is made up of a series of filesystem layers. Each layer adds, removes, or modifies files from the preceding layer in the filesystem. During runtime, there are a variety of different concrete implementations of such filesystems, including aufs , overlay , and overlay2.

Container images are typically combined with a container configuration file, which provides instructions on how to set up the container environment and execute an
application entry point. The container configuration often includes information on how to set up networking, namespace isolation, resource constraints (cgroups),
and what syscall restrictions should be placed on a running container instance. The container root filesystem and configuration file are typically bundled using the Docker image format.

Containers fall into two main categories:
- System containers
- Application containers


## Building Application Images with Docker

In general, container orchestration systems like Kubernetes are focused on building and deploying distributed systems made up of application containers.

### Dockerfiles

A Dockerfile can be used to automate the creation of a Docker container image.

#### Optimizing Image Sizes
The first thing to remember is that files that are removed by subsequent layers in the system are actually still present in the images; they’re just inaccessible.
Consider the following situation:

- layer A: contains a large file named 'BigFile'
    - layer B: removes 'BigFile'
        - layer C: builds on B by adding a static binary

You might think that BigFile is no longer present in this image. After all, when you run the image, it is no longer accessible. But in fact it is still present in layer A, which means that whenever you push or pull the image, BigFile is still transmitted through the network, even if you can no longer access it.

Another pitfall revolves around image caching and building. Remember that each layer is an independent delta from the layer below it. Every time you change a layer, it changes every layer that comes after it. Changing the preceding layers means that they need to be rebuilt, repushed, and repulled to deploy your image to development.

#### Image Security

When building images that will ultimately run in a production Kubernetes cluster, be sure to follow best practices for packaging and distributing applications. For example, don’t build containers with passwords baked in—and this includes not just in the final layer, but any layers in the image. Secrets and images should never be mixed. Depending on the language, you can achieve very small images with a very tight set of dependencies. This smaller set ensures that your image isn’t
exposed to vulnerabilities in libraries it would never use.

#### Multistage Image Builds

With multistage builds, rather than producing a single image, a Docker file can actually produce multiple images. Each image is considered a stage. Artifacts can be copied from preceding stages to the current stage. Building a container image using multistage builds can reduce your final container image size by hundreds
of megabytes and thus dramatically speed up your deployment times, since generally, deployment latency is gated on network performance.

#### Storing Images in a Remote Registry

Public registries are great for sharing images with the world because they allow for easy, unauthenticated use of the container images. You can easily distribute your software as a container image and have confidence that users everywhere will have the exact same experience.

In contrast, a private registry is best for storing applications that are private to your service and that you don’t want the world to use. Additionally, private registries often provide better availability and security guarantees because they are specific to you and your images rather than serving the world.

#### The Container Runtime Interface

Kubernetes provides an API for describing an application deployment, but relies on a container runtime to set up an application container using the container-specific APIs native to the target OS. On a Linux system that means configuring cgroups and namespaces. The interface to this container runtime is defined by the Container Runtime Interface (CRI) standard.