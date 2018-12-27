# Cloud  Agnostic Kubernetes Setup
System (CentOS 7.5)
* Master nodes
    - master01  - 172.16.1.211
    - master02  - 172.16.1.212
    - master03  - 172.16.1.213
* Worker nodes
    - node01    - 172.16.1.214
    - node02    - 172.16.1.215
    - node03    - 172.16.1.216
* Workstation
    - wks01     - 172.16.1.210
---
Setup steps:
1. Kubernetes (kubespray)
2. Istio
3. Rook
---
Setup kubectl
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
yum install -y kubectl
```
Copy admin.conf from master node
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```  