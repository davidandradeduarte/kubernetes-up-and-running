# Create kubernetes IN docker (kind) multi node cluster with

https://github.com/kubernetes-sigs/kind

```bash
kind create cluster --wait 5m
kubectl cluster-info
kind delete cluster
```