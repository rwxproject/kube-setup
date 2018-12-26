# Kubernetes Setup (kubespray)

On workstation - wrk01$

Generate ssh key
```
ssh-keygen -t rsa
```
Copy ssh key in each machine
```
NODES=(172.16.1.211 172.16.1.212 172.16.1.213 172.16.1.214 172.16.1.215 172.16.1.216)

for NODE in ${NODES[@]}
do
  ssh-copy-id -i ~/.ssh/id_rsa.pub root@$NODE
done
```
Type exit in each machine
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
```
sudo yum install -y ansible epel-release python-pip
sudo pip install Jinja
sudo yum install -y python-netaddr vim wget
```