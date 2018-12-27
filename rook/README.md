# Rook Ceph Setup
Download Rook
```
git clone https://github.com/rook/rook.git
cd rook/cluster/examples/kubernetes/ceph
kubectl apply -f operator.yaml
kubectl apply -f cluster.yaml
```
Verify rook ceph status
```
kubectl get all -n rook-ceph-system
kubectl get pod -n rook-ceph
```
Create storage class
```
kubectl apply -f storageclass.yaml
```
* Enable Toolbox
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