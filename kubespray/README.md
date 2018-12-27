# Kubernetes Setup (kubespray)

On workstation - wrk01$

Generate ssh key
```
ssh-keygen -t rsa
```
Copy ssh key in each machine, type yes and password
```
NODES=(172.16.1.211 172.16.1.212 172.16.1.213 172.16.1.214 172.16.1.215 172.16.1.216)

for NODE in ${NODES[@]}
do
  ssh-copy-id -i ~/.ssh/id_rsa.pub root@$NODE
done
```
Type exit in each machine (optional)
```
for NODE in ${NODES[@]}; do ssh root@$NODE; done
```
Enable wheel no password
```
sudo visudo
# uncomment the following line:
%wheel  ALL=(ALL)       NOPASSWD: ALL
```
Install packages
Note. Do not upgrade pip "'pip install --upgrade pip'"
```
// sudo yum install -y ansible 
sudo yum -y install epel-release
sudo yum -y install python-pip
// sudo pip install Jinja
// sudo yum install -y python-netaddr vim wget
```
Create cluster.inventory file:
```
vim cluster.inventory

declare -a IPS=(172.16.1.211 172.16.1.212 172.16.1.213 172.16.1.214 172.16.1.215 172.16.1.216)
CONFIG_FILE=inventory/appcluster/hosts.ini python3 contrib/inventory_builder/inventory.py ${IPS[@]}

***

ansible all -i inventory/appcluster/hosts.ini -m shell -a \
  'sudo sysctl -w net.ipv4.ip_forward=1'

ansible all -i inventory/appcluster/hosts.ini -m shell -a \
  'sudo systemctl stop firewalld && \
   sudo systemctl disable firewalld && \
   sudo systemctl mask firewalld && \
   sudo yum remove -y firewalld'

ansible all -i inventory/appcluster/hosts.ini -m shell -a \
  'SWAP=$(cat /etc/fstab | grep swap | sed "s/ .*//") && \
   sudo swapoff -v $SWAP && \
   sudo sed -i "/swap/d" /etc/fstab && \
   sudo rm $SWAP'

***

ansible-playbook -i inventory/appcluster/hosts.ini cluster.yml -b -v \
  --private-key=~/.ssh/private_key


---
master01 ansible_ssh_host=172.16.1.211
master02 ansible_ssh_host=172.16.1.212
master03 ansible_ssh_host=172.16.1.213
node01 ansible_ssh_host=172.16.1.214
node02 ansible_ssh_host=172.16.1.215
node03 ansible_ssh_host=172.16.1.216

[kube-master]
master01
master02
master03

[kube-node]
node01
node02
node03

[etcd] # Must be odd
master01
master02
master03

[k8s-cluster:children]
kube-node
kube-master
---
```
Setup cluster nodes
```
ansible all -i cluster.inventory -m shell -a \
  'sudo sysctl -w net.ipv4.ip_forward=1'

ansible all -i cluster.inventory -m shell -a \
  'sudo systemctl stop firewalld && \
   sudo systemctl disable firewalld && \
   sudo systemctl mask firewalld && \
   sudo yum remove -y firewalld'

ansible all -i cluster.inventory -m shell -a \
  'SWAP=$(cat /etc/fstab | grep swap | sed "s/ .*//") && \
   sudo swapoff -v $SWAP && \
   sudo sed -i "/swap/d" /etc/fstab && \
   sudo rm $SWAP'
```
Select kubespray release
https://github.com/kubernetes-incubator/kubespray/releases

Download kubespray
```
wget https://github.com/kubernetes-incubator/kubespray/archive/v2.7.0.tar.gz
tar -zxvf v2.7.0.tar.gz
rm -rf v2.7.0.tar.gz
mv kubespray-2.7.0/ kubespray
mv cluster.inventory kubespray/
```
Install python requirements
```
cd kubespray
pip install -r requirements.txt
```
Run Ansible playbook
```
F1=roles/kubernetes/master/defaults/main.yml
sed -i '/ResourceQuota/ a \  - Priority' $F1
sed -i '/registr/ a \  - scheduling.k8s.io/v1alpha1=true' $F1
F2=roles/kubespray-defaults/defaults/main.yaml
sed -i "/kube_feature/ s/'\]/', 'PodPriority=true']/" $F2

ansible-playbook \
  -i cluster.inventory \
  --flush-cache \
  cluster.yml \
  -b -v \
  --private-key=~/.ssh/id_rsa
```