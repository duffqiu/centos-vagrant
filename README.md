# centos-vagrant
vagrant file to build a kubernets cluster which container 1 master and 3 nodes. You done't need to create complicated ca files and do a lot of configuration

### Why don't do that with kubeadm

Because I want to setup the etcd, apiserver, controller, scheduler without docker container

### Architecutre

![archi](https://github.com/duffqiu/centos-vagrant/blob/master/pic/arch.png)


### Cluster network
The default setting will create the private network from 172.17.8.101 to 172.17.8.103 for nodes, and it will use the host's dhcp for the public ip

The kubenetes service's vip range is 10.254.0.0/16

The container network range is 170.30.0.0/30 owned by flanneld with GW mode

### Usage

#### Prerequisite
* Host server with 8G+ mem(More is better), 60G disk, 8 core cpu at lease
* vagrant 2.0+
* virtualbox 5.0+
* Maybe need to access the internet through greate firewall to download the kubernetes files

#### Setup
```
git clone duffqiu/centos-vagrant

cd centos-vagrant

vagrant up
```

#### Connect to kubernetes

```
vagrant ssh node1
sudo su
kubectl get nodes
```



