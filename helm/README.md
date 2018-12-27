# Helm setup
* Client
```
wget https://storage.googleapis.com/kubernetes-helm/helm-v2.12.1-linux-amd64.tar.gz
tar -zxvf helm-v2.12.1-linux-amd64.tar.gz 
mv linux-amd64/helm /usr/local/bin/helm
```
* Cluster
```
helm init
```