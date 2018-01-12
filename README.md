# centos-vagrant
vagrant file to build a kubernets cluster which container 1 master and 3 nodes

### Architecutre

![archi]()


### Cluster network
The default setting will create the private network from 172.17.8.101, and it will use the host's dhcp for the public ip
The kubenetes service's vip range is 10.254.0.0/16

### Usage

#### Prerequisite
* vagrant
* virtualbox
* Maybe need to access the internet through greate firewall to download the kubernetes files

#### Setup
```
git clone duffqiu/centos-vagrant

cd centos-vagrant

vagrant up
```



