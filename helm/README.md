# Helm setup

- Client

```
wget https://storage.googleapis.com/kubernetes-helm/helm-v2.14.2-linux-amd64.tar.gz
tar -zxvf helm-v2.14.2-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/helm
```

- Cluster

```
vim rbac-config.yaml
```

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

# install tiller

```
kubectl create -f rbac-config.yaml
helm init --service-account tiller --history-max 200
```

# install helm repos

```
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo add confluent https://confluentinc.github.io/cp-helm-charts/
```
