# Utils (CentOS)
* Setup fixed ip address
```
sudo vi /etc/sysconfig/network-scripts/ifcfg-ens32 

BOOTPROTO="static"
...
IPADDR=172.16.1.213
NETMASK=255.255.255.0
GATEWAY=172.16.1.1
DNS1=8.8.8.8
DNS2=8.8.4.4
```

kubeadm token create --print-join-command

kubectl cordon node2
kubectl uncordon node2

linux network tools
https://github.com/nicolaka/netshoot