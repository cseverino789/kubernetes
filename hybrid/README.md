# Deploy the Storage Server - PV (hybrid installations, persistent volume)

For hybrid installations, the Storage Server can be installed in your cluster.

*This example stores data on a volume in your cluster. To store data in S3, see [Deploy the Storage Server - S3](s3)*

The Storage Server [Dockerfile](Dockerfile) is based on the [Installing the Storage Server on a Local Server](https://doc.overops.com/docs/installing-the-storage-server-on-a-local-server) guide, with some minor modifications.

For complete instructions on performing a hybrid installation, refer to the [Hybrid Installation on Linux](https://doc.overops.com/docs/linux-hybrid-installation) guide.

The file `settings.yaml` must be mounted into the `/opt/takipi-storage/private` directory to run this container. An example [settings.yaml](private/settings.yaml) can be found in this repo.

## Quick Start

This image is on Docker Hub: [overops/storage-server](https://hub.docker.com/r/overops/storage-server)

### Docker Quick Start

```console
docker run -d -p 8080:8080 -p 8081:8081 --mount type=bind,source="$(pwd)"/storage,target=/opt/takipi-storage/storage  --mount type=bind,source="$(pwd)"/private,target=/opt/takipi-storage/private overops/storage-server
```

### Kubernetes Quick Start

```console
kubectl create secret generic overops-storage --from-file=./settings.yaml
kubectl apply -f https://raw.githubusercontent.com/takipi-field/kubernetes/master/hybrid/overops-storage-server.yaml
```

## Build

If you're deploying this image locally on a Minikube cluster, first set the Docker environmental variables:

```console
eval $(minikube docker-env)
```

Build the image and tag it `overops-storage-server`.

```console
docker build . -t overops-storage-server
```

## Run in Docker

This container expects a `private` directory containing `settings.yaml` to be mounted into the `/opt/takipi-storage/private` directory. When running the container, we'll mount the `private` directory into the container.

To persist data for longer than the lifetime of a container, mount a volume to the `/opt/takipi-storage/storage` directory.
We'll also forward port 8080 (application) and 8081 (admin) from the host machine to the container.

```console
docker run -p 8080:8080 -p 8081:8081 --mount type=bind,source="$(pwd)"/storage,target=/opt/takipi-storage/storage --mount type=bind,source="$(pwd)"/private,target=/opt/takipi-storage/private overops-storage-server
```

For debugging, it can be useful to run the container interactively:

```console
$ docker run -it overops-storage-server /bin/bash
# ./run.sh
```

To run in the background:

```console
docker run -d -p 8080:8080 -p 8081:8081 --mount type=bind,source="$(pwd)"/storage,target=/opt/takipi-storage/storage --mount type=bind,source="$(pwd)"/private,target=/opt/takipi-storage/private overops-storage-server
```

## Run in Kubernetes

In Kubernetes, we'll store `setting.yaml` as a [Secret](https://kubernetes.io/docs/concepts/configuration/secret/).

Create the secret from local file:

```console
kubectl create secret generic overops-storage --from-file=./settings.yaml
```

Choose the [Volume](https://kubernetes.io/docs/concepts/storage/volumes/) type that makes the most sense for your cluster. When running locally in Minikube, we'll use [hostPath](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath).

Modify [`overops-storage-server.yaml`](overops-storage-server.yaml) as needed for your environment, then create the deployment and service:

```console
kubectl apply -f overops-storage-server.yaml
```

[Verifying Storage Server Installation](https://doc.overops.com/docs/verifying-storage-server-installation)

To verify when running in Minikube:

```console
$ curl -I http://$(minikube ip):30080/storage/v1/diag/ping
HTTP/1.1 200 OK
Date: Wed, 05 Dec 2018 15:55:05 GMT
Content-Type: text/plain
Content-Length: 2
```

To remove:

```console
kubectl delete -f overops-storage-server.yaml
```

### Note about Security Context

Starting with version 4.56.0, all published images to Docker hub are now "rootless." Images are now using the base image openjdk:8-jre-slim and running with user ID 1000 and group ID 1000 instead of the root user. This is now reflected in the kubernetes deployment's security context.