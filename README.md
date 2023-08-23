# DUCKS (Dynamic API for Unleashing Cloud KServe)
## _A walkthrough on creating ephemeral AI, GPU driven APIs in the Cloud_

//IMAGE TO BE CREATED UPON COMPLETION

![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)

![Alt Text](./media/images/real-mvp.gif)

### MVP: Minimal Viable Product

An API call should spin up the approrpiate amount of GPU compute for the single call and return an image, or a pointer to an image in S3.

You should be able to query the available models that are available (stored in S3) and include that in the API call.

# Walkthrough

Through this walkthrough, there will be use of some [DevOps Terminology](#devops-terminology) (this's a local link to the help).

Our goal is to deploy [KubeFlow](#kubeflow) onto different cloud providers to create ephemeral environments per API call with [serverless](#serverless) best practices in mind.

First we'll do it in our local environment:

## Deploy Locally

### Tech Stack (install these)

![Alt Text](./media/images/surprise-whats-in-the-box.gif)

- [Python 3.10+](https://www.python.org/downloads/): Logic/Code layer
- [Docker](#docker): [Containerization](#container) layer
- [MiniKube](https://minikube.sigs.k8s.io/docs/): Local [Kubernetes](#kubernetes) Cluster
- [Kubectl](https://kubernetes.io/docs/reference/kubectl/): [Kubernetes](#kubernetes) Command Line (CLI) Tool
- [KIND](https://kind.sigs.k8s.io): run local [Kubernetes](#kubernetes) cluster
- [KServe](#kserve): You must install these tools:
> If you start from scratch, KServe requires Kubernetes 1.17+, Knative 0.19+, Istio 1.9+.


<!-- 
- [go](https://go.dev/dl/): KServe controller is written in Go and requires Go 1.18.0+.
- git: For source control.
- Go Module: Go's new dependency management system.
- ko: For development.
- kubectl: For managing development environments.
- kustomize To customize YAMLs for different environments, requires v3.5.4+.
- yq yq is used in the project makefiles to parse and display YAML output requires yq 4.*. -->

<!-- ```shell
$ go version
go version go1.21.0 windows/386
$ git version 
git version 2.39.1.windows.1
``` -->

## HOW R HAXOR!?
![Alt Text](./media/images/swordfish-hack.gif)

Create a local Kubernetes environment with:

```shell
kind create cluster --name=local-kserve
``` 

Now that we've created a new cluster locally in Docker, we should be able to see it with:
```shell
cat ~/.kube/config
```

This should return something like:
```shell
Creating cluster "local-kserve" ...
 • Ensuring node image (kindest/node:v1.27.3) 🖼  ...
 ✓ Ensuring node image (kindest/node:v1.27.3) 🖼
 • Preparing nodes 📦   ...
 ✓ Preparing nodes 📦 
 • Writing configuration 📜  ...
 ✓ Writing configuration 📜
 • Starting control-plane 🕹️  ...
 ✓ Starting control-plane 🕹️
 • Installing CNI 🔌  ...
 ✓ Installing CNI 🔌
 • Installing StorageClass 💾  ...
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-local-kserve"
You can now use your cluster with:

kubectl cluster-info --context kind-local-kserve

Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 🙂
```

To confirm you have the correct context, run:
```shell
kubectl config get-contexts
```
This should return something like:

```shell
CURRENT   NAME                CLUSTER             AUTHINFO                     NAMESPACE
*         kind-local-kserve   kind-local-kserve   kind-local-kserve
```

Now that we have a context referencing our local cluster, we need a namespace:
```shell
kubectl create namespace kserve-test
```
Once we've created the namespace we'll want to set it to our current context:

```shell
kubectl config set-context --current --namespace kserve-test
kubectl config get-contexts
```

Now your namespace should be defined for your current context:

```shell
CURRENT   NAME                CLUSTER             AUTHINFO                     NAMESPACE
*         kind-local-kserve   kind-local-kserve   kind-local-kserve            kserve-test
```


Before we can run the InferenceService on KServe we have to install the quickstart services onto KIND. First lets use Docker Exec to open a terminal in our KIND (K8s in Docker) environment:

```shell
$ docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED        STATUS        PORTS                       NAMES
<CONTAINER_ID>   kindest/node:v1.27.3   "/usr/local/bin/entr…"   15 hours ago   Up 15 hours   127.0.0.1:56227->6443/tcp   local-kserve-control-plane
```
Now that we know the ID for our KIND container, we'll Docker Exec into the container and run the quickstart install from [KServe Quickstart Guiide](https://kserve.github.io/website/0.10/get_started/#install-the-kserve-quickstart-environment) and be sure to exit the container's bash before continuing: 

```shell
terminal: $ docker exec -it <CONTAINER_ID> /bin/bash
root@local-kserve-control-plane: $ curl -s "https://raw.githubusercontent.com/kserve/kserve/release-0.10/hack/quick_install.sh" | bash
root@local-kserve-control-plane: $ exit
```
[KServe Quickstart Guide](https://kserve.github.io/website/0.10/get_started/#before-you-begin)
Now take the CONTAINER ID and use the following:

In Kubernetes, the [CRDs](#custom-resource-definitions-crds) (Custom Resource Definitions) are installed on the Kubernetes controller (specifically within the API server) so that it understands and recognizes the custom resources you define.

The controller acts as the central brain of the Kubernetes cluster. It continuously monitors the desired state of your resources and takes actions to maintain that desired state. When a new custom resource is introduced via the [CRD](#custom-resource-definitions-crds), the controller learns about it and establishes the corresponding API endpoints to manage and interact with these resources.

With the CRDs installed on the controller, the pods (running applications or containers) can then interact with the Kubernetes API server to create, update, or delete instances of the custom resources. This allows the pods to utilize the Kubernetes control plane for managing and orchestrating these custom resources.

By defining your own custom resources using CRDs, you can extend the Kubernetes API and leverage Kubernetes' robust infrastructure management capabilities for your specific application needs.

It's worth noting that CRDs do not directly provide network routing functionality, but rather enable you to define and manage custom resources that may interact with the networking components of your Kubernetes cluster, such as ingress controllers or service meshes.

I hope this clarifies the relationship between CRDs, pods, and the controller within Kubernetes. Let me know if you have further questions!

Clone this repo and run 



## Clouds
These DUCKS should be able to fly in any cloud. We'll start with [CoreWeave](https://www.coreweave.com).


We've only ever flown in AWS so for now, this is the "North Star" to guide the flight. New diagram to come.

![Alt Text](./media/diagrams/mvp.png)

## DevOps Terminology

### Container

Think of a container like a shipping container for software. Docker is a company that makes standardized shipping containers. As a shipping container can be deployed on a train, truck, or boat because of it's standardization; in software, a container can be deployed on localhost, AWS, or CoreWeave just as easily and is operating system independent. It's effectively a virtualization layer that's extremely portable.

### Docker

[Docker](https://www.docker.com) is to containers as Kleenex of tissue paper. They're the first containerization platform you think of and, as such, have cornered the market. They've recently become less Open Source minded. So I hope to eventually move most of my projects to [Open Container Initiative](https://opencontainers.org)

### Kubernetes

[Kubernetes](https://kubernetes.io/docs/concepts/overview/), also known as K8s, is an open-source system for automating deployment, scaling, and management of containerized applications.

#### Control Plane

The control plane is responsible for managing and controlling the Kubernetes cluster. It consists of various components, such as the API server, controller manager, scheduler, and etcd. These components work together to maintain the desired state of the cluster and handle tasks like scheduling pods, scaling resources, and managing networking.

#### Context

In Kubernetes, a context is a set of access parameters that determine the cluster, user, and namespace to use when interacting with the cluster. It includes information like the cluster's server address, authentication credentials, and the default namespace. Contexts allow users to switch between different clusters or namespaces without having to reconfigure their command-line tools.

#### Cluster

A cluster in Kubernetes refers to a set of physical or virtual machines, called nodes, that work together to run containerized applications. It includes the control plane components and worker nodes. The control plane manages the cluster, while the worker nodes host the running containers.

### KServe

[KServe](https://github.com/kserve/kserve) provides a Kubernetes [Custom Resource Definition](#custom-resource-definitions-crds) for serving machine learning (ML) models on arbitrary frameworks. It aims to solve production model serving use cases by providing performant, high abstraction interfaces for common ML frameworks like Tensorflow, XGBoost, ScikitLearn, PyTorch, and ONNX.

It encapsulates the complexity of autoscaling, networking, health checking, and server configuration to bring cutting edge serving features like GPU Autoscaling, Scale to Zero, and Canary Rollouts to your ML deployments. It enables a simple, pluggable, and complete story for Production ML Serving including prediction, pre-processing, post-processing and explainability. KServe is being used across various organizations.

### KubeFlow

The [KubeFlow](https://www.kubeflow.org) project is dedicated to making deployments of machine learning (ML) workflows on Kubernetes simple, portable and scalable. Our goal is not to recreate other services, but to provide a straightforward way to deploy best-of-breed open-source systems for ML to diverse infrastructures. Anywhere you are running Kubernetes, you should be able to run Kubeflow.

### Serverless

- Serverless doesn't mean there are no servers. It means the complexity of server management and capacity planning is abstracted away from developers.
- It's synonymous with auto-scaling to the extent that serverless solutions can scale from handling zero requests (hence, no cost) to handling thousands or millions, based on demand.
- The beauty of serverless is its economic model: when there's no traffic, there's no cost.


### Custom Resource Definitions (CRDs)

A [Custom Resource Definition](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) is a reference to an external API that deviates from Kubernetes core functionality. In our case, InferenceService is a kind (not to be confused with K8s in Docker) that is only defined in the API for _serving.kserve.io/v1beta1_. We will need to load the CRDs before we are able to reference components of KServe.


------

## Brain Dump (Wants)

![Alt Text](./media/images/ThisIsFine.jpeg)

- S3 Front End
- CloudFront
- Website


## Backlog

- Discord Bot
- Twitch Bot

---
![Alt Text](./media/images/dumpsterfire-dumpster.gif)

## References Credited
- [DagsHub: How To Install Kubeflow Locally](https://dagshub.com/blog/how-to-install-kubeflow-locally/)
- [KubeFlow](https://www.kubeflow.org)
