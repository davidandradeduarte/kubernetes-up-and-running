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

### Cluster Components

- **Kubernetes Proxy** - responsible for routing network traffic to load-balanced services in the Kubernetes cluster. It's present on every node in the cluster. Usually runs as a DaemonSet object.
    ```bash
    kubectl get daemonSets --namespace=kube-system kube-proxy
    ```

- **Kubernetes DNS** - provides naming and discovery for the services that are defined in the cluster. Runs as a replicated service on the cluster. Depending on the size of the cluster, there may be one or more. Runs as a Deployment object.
    ```bash
    kubectl get deployments --namespace=kube-system core-dns
    ```
    There's also a Service object that provides load balancing for the DNS server.
    ```bash
    kubectl get services --namespace=kube-system core-dns
    ```

- **Kubernetes UI** - kubernetes GUI application. A web dashboard. Runs as a single replica and it's a Deployment object.
    ```bash
    kubectl get deployments --namespace=kube-system kubernetes-dashboard
    ```
    Also has a service that performs load balancing for the dashboard:
    ```bash
    kubectl get services --namespace=kube-system kubernetes-dashboard
    ```
    Access the kubernetes dashboard:
    ```bash
    kubectl proxy
    ```
    Some providers don’t install the Kubernetes dashboard by default.

## Chapter 4. Common kubectl Commands

The kubectl tool is a CLI to create objects and interact with the Kubernetes API.

### Namespaces

Organizes objects in the cluster.

`default` namespace is selected by default.

Select a different namespace with the `--namespace`, or short `-n` flag.

### Contexts

Can save configuration for default namespace, how to both find and authen‐ ticate to clusters, etc

Context configurations is stored in `$HOME/.kube/config`

Set default namespace to `mystuff` in context `my-context`:
```bash
kubectl config set-context my-context --namespace=mystuff
```

Use context `my-context`:
```bash
kubectl config use-context my-context
```

### Viewing Kubernetes API Objects

Everything contained in Kubernetes is represented by a RESTful resource - Kubernetes objects.

Each Kubernetes object exists at a unique HTTP path. E.g `https://your-k8s.com/api/v1/name‐spaces/default/pods/my-pod`

Get a kubernetes object:
```bash
kubectl get <resource-name>
kubectl get pod
```

The `-o` flag  manipulates output format. E.g `-o wide` gives more information. `-oyaml` outputs the resource in yaml format.

You can use JSONPath query language.  
*E.g Extracts the ip of the specified pod*:
```bash
kubectl get pods my-pod -o jsonpath --template={.status.podIP}
```

More detailed information about a particular object:
```bash
kubectl describe <resource-name> <obj-name>
```

### Creating, Updating, and Destroying Kubernetes Objects

Create a kubernetes object (type is infered by the manifest):
```bash
kubectl apply -f obj.yaml
```

Use `--dry-run` flag to see what the apply command will do without actually making the changes.

Edit a kubernetes object:
```bash
kubectl edit <resource-name> <obj-name>
```

Delete a kubernetes object:
```bash
kubectl delete -f obj.yaml
```

### Labeling and Annotating Objects

Label a kubernetes object:
```bash
kubectl label pods bar color=red
```

Remove label from kubernets object:
```bash
kubectl label pods bar color-
```

### Debugging Commands

Logs of a running container:
```bash
kubectl logs <pod-name>
```
Use the `-c` flag to set the container (in case you have multiple containers inside a Pod)  
Use the `-f` (follow) flag to keep logs open (e.g tail -f)

Execute a command in a running container:
```bash
kubectl exec -it <pod-name> -- bash
```

Copy a file from a running container to local machine:
```bash
kubectl cp <pod-name>:</path/to/remote/file> </path/to/local/file>
```

Forward network traffic from the local machine to the Pod:
```bash
kubectl port-forward <pod-name> 8080:80
```

Nodes/Pods resource usage:
```
kubectl top nodes
kubectl top pods
```

### Command Autocompletion

https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/#enable-shell-autocompletion

### Alternative Ways of Viewing Your Cluster

- [Visual Studio Code Kubernetes extension](https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools)
- [Jetbrains extension for their IDEs](https://plugins.jetbrains.com/plugin/10485-kubernetes)
- [Mobile open source app](https://github.com/bitnami-labs/cabin)

Although not mentioned in the book:
- [Lens](https://k8slens.dev/)
- [k9s](https://github.com/derailed/k9s)

## Chapter 5. Pods

A Pod represents a collection of application containers and volumes running in the same execution environment.

`Pod` is the smallest deployable artifact in a Kubernetes cluster.

Applications running in the same Pod share the same IP address and port space, have the same hostname 

*The name goes with the whale theme of Docker containers, since a Pod is also a group of whales*

Usually the Pod runs a main application container and we put *sidecar* containers within it that extend and enhances the functionality of the main container. For example, several Service Mesh implementations use sidecars to inject network management into an application’s Pod.

Example of a Pod manifest:  
[playground/manifests/kuard-pod.yaml](playground/manifests/kuard-pod.yaml)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:blue
      name: kuard
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP

```

Create a pod:
```bash
kubectl apply -f manifests/kuard-pod.yaml
```

Get a pod:
```bash
kubectl get pods kuard
```

Possible Pod statuses: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/

Additional Pod information:
*(this will show event streams - not attached to the Pod object)*
```bash
kubectl describe pods kuard
```

Delete a pod:
```bash
kubectl delete pods/kuard
# using the same file that was used to create it:
kubectl delete -f kuard-pod.yaml
```

When a Pod is deleted it will enter the `Terminating` state for a termination grace period.

Port Forwarding a Pod:
```bash
kubectl port-forward kuard 8080:8080
```

View Pod logs:
```bash
kubectl logs kuard
kubectl logs -f kuard # continuously stream logs
kubectl logs -p kuard # see logs from previous instance
```

Execute commands or "bash into" a Pod:
```bash
kubectl exec -it kuard -- bash
# we can pass any command besides bash e.g:
kubectl exec kuard -- date
```

Copying files into a container is an anti-pattern. Treat the contents of a container as immutable.

### Healt checks

#### Liveness Probe

Liveness determines if an application is running properly.

Containers that fail liveness checks are restarted, by default.

Liveness health checks are application-specific, defined in the Pod manifest.

Liveness probes are defined per container, which means each container inside a Pod is health-checked separately.

[kuard-pod-health.yaml](playground/manifests/kuard-pod-health.yaml)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:blue
      name: kuard
      livenessProbe: # liveness health check
        httpGet:
          path: /healthy
          port: 8080
        initialDelaySeconds: 5 # after 5 seconds from startup the endpoint will be called
        timeoutSeconds: 1 # should respond in within 1 second 
        periodSeconds: 10 # call this endpoint every 10 seconds
        failureThreshold: 3 # if it fails 3 consecutive times, it will restart
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
```

#### Readiness Probe

Readiness describes when a container is ready to serve user requests.

Containers that fail readiness checks are removed from service load balancers.

Readiness probes are config‐ ured similarly to liveness probes.

#### Types of Health Checks

There's also the possibility of configuring non-HTTP health checks like TCP socket connections (`tcpSocket`).

`exec` probes execute a script or program in the context of the container. If exit code is 0 succeeds, otherwise fails.

## Side Notes

`etcd` stores the object manifests

Good description about how the Kubernetes API and scheduler manage and deploy Pods:  
*The Kubernetes API server accepts and processes Pod manifests before storing them in persistent storage (etcd). The scheduler also uses the Kubernetes API to find Pods that haven’t been scheduled to a node. The scheduler then places the Pods onto nodes depending on the resources and other constraints expressed in the Pod manifests. Multiple Pods can be placed on the same machine as long as there are sufficient resources. However, scheduling multiple replicas of the same application onto the same machine is worse for reliability, since the machine is a single failure domain. Consequently, the Kubernetes scheduler tries to ensure that Pods from the same application are distributed onto different machines for reliability in the presence of such failures. Once scheduled to a node, Pods don’t move and must be explicitly destroyed and rescheduled.*