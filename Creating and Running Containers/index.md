# Creating and Running Containers

Container images bundle a program and its dependencies into a single artifact under a root filesystem. The most popular container image format is the Docker image
format, which has been standardized by the Open Container Initiative to the OCI image format. Kubernetes supports both Docker- and OCI-compatible images via
Docker and other runtimes.

## Container images

A container image is a binary package that encapsulates all of the files necessary to run a program inside of an OS container. The most popular and widespread container image format is the Docker image format, which was developed by the Docker open source project for packaging, distributing, and running containers using the docker command. Docker image format continues to be the de facto standard and is made up of a series of filesystem layers. Each layer adds, removes, or modifies files from the preceding layer in the filesystem. During runtime, there are a variety of different concrete implementations of such
filesystems, including aufs , overlay , and overlay2.

Container images are typically combined with a container configuration file, which provides instructions on how to set up the container environment and execute an
application entry point. The container configuration often includes information on how to set up networking, namespace isolation, resource constraints (cgroups),
and what syscall restrictions should be placed on a running container instance. The container root filesystem and configuration file are typically bundled using the Docker image format.

Containers fall into two main categories:
- System containers
- Application containers


## Building Application Images with Docker