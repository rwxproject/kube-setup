To install MetalLB, apply the manifest:

```shell
kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.8.3/manifests/metallb.yaml
```

Layer2 configuration

```shell
kubectl apply -f layer2-configmap.yaml
```

Example

```shell
kubectl apply -f app-example.yaml
```
