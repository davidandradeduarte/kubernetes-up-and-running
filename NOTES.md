# Notes

## Chapter 1. Introduction

Kubernetes is an orchestrator for deploying containerized applications.

Key reasons to use Kubernetes:
- Velocity
- Scaling (of both software and teams)
- Abstracting your infrastructure
- Efficiency
- Immutability
- Declarative configuration
- Online self-healing systems

## Chapter 2. Creating and Running Containers

Containers isolate an application to run on an immutable operating system with their own libs, dependencies and source code, which makes applications portable to any guest or infrastructure as long as there is a contaiter runtime system.

Image registries are available in all major cloud providers and make it easy for users to manage and deploy private images, while image-builder services provide easy integration with continuous delivery systems.

Docker image/containers gotchas:
- Docker images are divided into multiple layers
- Order your layers from least likely to change to most likely to change in order to optimize the image size for pushing and pulling (e.g dependencies should come before source code because source codes changes more frequently than dependencies)
- Multistage builds can significantly reduce the size of a docker image if we only copy the compiled binary and run it in the last stage

Kubernetes uses the `kubelet` daemon to launch containers.

## Chapter 3. Deploying a Kubernetes Cluster

We can install Kubernetes on any major cloud provider as a service (*KaaS - Kubernetes as a Service*) or configure it to run on bare metal.  
- [Azure](playground/AKS.md) - Azure Kubernetes Service (AKS)
- Google - Google Kubernetes Engine (GKE)
- AWS - Elastic Kubernetes Service (EKS)

There are also tools to install and configure Kubernetes on your local machine using a virtualization platform or Docker (recommended for learning or testing purposes):
- [minikube](playground/minikube.md) - most recently they added Docker support, but it can be installed on a virtualization platform as well (e.g `virtualbox`)
- [kind](playground/kind.md) - kubernetes in docker

There are many others like microk8s and k3d, but the book only references these two.

## Chapter 4. Common kubectl Commands

## Chapter 5. Pods

