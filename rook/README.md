# Rook Ceph Setup

Install Rook-Ceph helm chart

```
helm repo add rook-release https://charts.rook.io/release
helm install --namespace rook-ceph rook-release/rook-ceph
```

Download Rook

```
# git clone https://github.com/rook/rook.git
# kubectl apply -f rook/cluster/examples/kubernetes/ceph/operator.yaml
# kubectl apply -f rook/cluster/examples/kubernetes/ceph/cluster.yaml
```

Verify rook ceph status

```
# kubectl get all -n rook-ceph-system
kubectl get pod -n rook-ceph
```

Create storage class

```
kubectl apply -f rook/cluster/examples/kubernetes/ceph/storageclass.yaml
```

- Enable Toolbox

```
kubectl apply -f toolbox.yaml
```

Ceph status

```
kubectl -n rook-ceph exec -it rook-ceph-tools-< xxx > bash
ceph status
ceph osd status
ceph df
rados df
exit
```

MySQL example

```
kubectl apply -f mysql-demo.yaml
kubectl get pvc,pv,svc,pod,deploy,rs
```

Confirm inside pod

```
kubectl exec -it demo-mysql-7fbfff59d9-94kkm -- bash
ls -la /var/lib/mysql
mysql -u root -pchangeme
> create database demo; use demo; create table helloworld (id int, hello varchar(100)); insert into helloworld values(1,'world');
> select * from helloworld;
> \q
exit
```

- Object Storage

```
kubectl apply -f rook/cluster/examples/kubernetes/ceph/object.yaml
```

Create s3 user

```
kubectl -n rook-ceph exec -it rook-ceph-tools-76c7d559b6-ncz6f -- bash
radosgw-admin user create --uid rook-user --display-name "A rook rgw User" --rgw-realm=my-store --rgw-zonegroup=my-store
...// copy keys
// from service . namespace (CLUSTER-IP)
export AWS_HOST=service/rook-ceph-rgw-my-store.rook-ceph
// from service ip address
export AWS_ENDPOINT=10.233.10.161
// from json output
export AWS_ACCESS_KEY_ID=X81FGARON3YZEXR97GR4
export AWS_SECRET_ACCESS_KEY=7turVaKLAuTmdfFxHSOtRLYgfDGiXg1ee2LXEwZe
```

S3 commands - http://s3tools.org

```
s3cmd --help
s3cmd mb --no-ssl --host=${AWS_HOST} --host-bucket= s3://demobucket
```

- File Storage

```
kubectl apply -f rook/cluster/examples/kubernetes/ceph/filesystem.yaml
```
