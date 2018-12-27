# Kubernetes Setup (kubespray)

wrk1$ (172.16.1.210)

Generate ssh key
```
ssh-keygen -t rsa
```
Copy ssh key in each machine, type yes and password
```
declare -a IPS=(172.16.1.211 172.16.1.212 172.16.1.213)
for IP in ${IPS[@]}
do
  ssh-copy-id -i ~/.ssh/id_rsa.pub root@$IP
done
```
Type exit in each machine (optional) - no password 
```
for IP in ${IPS[@]}; do ssh root@$IP; done
```
Install packages
Note. Do not upgrade pip "'pip install --upgrade pip'"
```
sudo yum -y install epel-release
sudo yum -y install python-pip
sudo yum -y install git vim wget

sudo yum -y install centos-release-scl
sudo yum -y install rh-python36
scl enable rh-python36 bash
sudo yum -y groupinstall 'Development Tools'
```
Download kubespray and install requirements
```
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray
sudo pip install -r requirements.txt
```
Configure nodes file (hosts.ini)
```
cp -rfp inventory/sample inventory/appcluster
declare -a IPS=(172.16.1.211 172.16.1.212 172.16.1.213)
CONFIG_FILE=inventory/appcluster/hosts.ini python3 contrib/inventory_builder/inventory.py ${IPS[@]}

cat inventory/appcluster/hosts.ini
```
Basic node config
```
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
```
Deploy Kubernetes
```
ansible-playbook -i inventory/appcluster/hosts.ini cluster.yml -b -v \
  --private-key=~/.ssh/private_key
```
Install kubectl
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
Access Kubernetes cluster configuration
(copy admin.conf from node)
```
mkdir -p $HOME/.kube
sudo scp root@172.16.1.211:/etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```  