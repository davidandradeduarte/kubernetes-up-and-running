# Create kubernetes single node cluster with minikube

https://github.com/kubernetes/minikube

```bash
minikube start # docker by default if installed
# minikube start --driver=virtualbox
kubectl get pods -n kube-system
minikube dashboard
minikube stop
```