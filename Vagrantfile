# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  #config.vm.box = "centos/7"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  config.vm.box_check_update = false

  $num_instances = 3

  # curl https://discovery.etcd.io/new?size=3
  $durl = "node1=http://172.17.8.101:2380,node2=http://172.17.8.102:2380,node3=http://172.17.8.103:2380"

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  (1..$num_instances).each do |i|

    config.vm.define "node#{i}" do |node|
    node.vm.box = "centos/7"
    node.vm.hostname = "centos#{i}"
    ip = "172.17.8.#{i+100}"
    node.vm.network "private_network", ip: ip
    node.vm.network "public_network", bridge: "en1: Wi-Fi (AirPort)", auto_config: false
    #node.vm.synced_folder "/Users/DuffQiu/share", "/home/vagrant/share"

    node.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
      vb.memory = "2048"
      vb.cpus = 2
      vb.name = "centos#{i}"
    end

    config.vm.provision "shell" do |s|
      s.inline = <<-SHELL
        cp /vagrant/yum/*.* /etc/yum.repos.d/
        yum install -y wget curl conntrack-tools
        
        echo 'disable selinux'
        setenforce 0
        sed -i 's/=enforcing/=permissive/g' /etc/selinux/config

        echo 'enable iptable kernel parameter'
        cat  <<EOF > /etc/sysctl.d/k8s.conf
        net.bridge.bridge-nf-call-ip6tables = 1
        net.bridge.bridge-nf-call-iptables = 1
        EOF

        echo 'set host name resolution'
        cat <<EOF >>/etc/hosts
        172.17.8.101 master
        172.17.8.101 node1
        172.17.8.102 node2
        172.17.8.103 node3
        EOF

        cat /etc/hosts

        echo 'disable swap'
        swapoff -a
        sed -i 's/\/dev\/mapper\/VolGroup00-LogVol01/#\/dev\/mapper\/VolGroup00-LogVol01/g' /etc/fstab
       

        #create group if not exists  
        egrep "^docker" /etc/group >& /dev/null  
        if [ $? -ne 0 ]  
        then  
          groupadd docker 
        fi

        usermod -aG docker vagrant
        rm -rf ~/.docker/
        systemctl stop docker
        systemctl disable flanneld
        yum install -y docker.x86_64
        
        echo { > /etc/docker/daemon.json
        echo '  "registry-mirrors" : ["http://2595fda0.m.daocloud.io"]' >> /etc/docker/daemon.json
        echo } >> /etc/docker/daemon.json  

        systemctl stop etcd
        yum install -y etcd

        
        echo '#[Member]' >/etc/etcd/etcd.conf
        echo 'ETCD_DATA_DIR="/var/lib/etcd/default.etcd"' >>/etc/etcd/etcd.conf
        echo 'ETCD_LISTEN_PEER_URLS="http://'$2':2380"' >>/etc/etcd/etcd.conf
        echo 'ETCD_LISTEN_CLIENT_URLS="http://'$2':2379,http://localhost:2379"' >>/etc/etcd/etcd.conf
        echo 'ETCD_NAME="node'$1'"' >>/etc/etcd/etcd.conf
        
        echo '#[Clustering]' >>/etc/etcd/etcd.conf
        echo 'ETCD_INITIAL_ADVERTISE_PEER_URLS="http://'$2':2380"' >>/etc/etcd/etcd.conf
        echo 'ETCD_ADVERTISE_CLIENT_URLS="http://'$2':2379"' >>/etc/etcd/etcd.conf
        echo 'ETCD_INITIAL_CLUSTER="'$3'"' >>/etc/etcd/etcd.conf
        echo 'ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"' >>/etc/etcd/etcd.conf
        echo 'ETCD_INITIAL_CLUSTER_STATE="new"' >>/etc/etcd/etcd.conf
        

        cat /etc/etcd/etcd.conf 
        sleep 5

        echo 'start etcd...'
        systemctl daemon-reload
        systemctl enable etcd 
        systemctl start etcd  >&- 

        echo 'install flannel...'
        systemctl stop flanneld
        systemctl disable flanneld
        yum install -y flannel

        echo 'create flannel config file...'
        echo '# Flanneld configuration options' > /etc/sysconfig/flanneld
        echo 'FLANNEL_ETCD_ENDPOINTS="http://localhost:2379"' >> /etc/sysconfig/flanneld
        echo 'FLANNEL_ETCD_PREFIX="/kube-centos/network"' >> /etc/sysconfig/flanneld
        echo 'FLANNEL_OPTIONS="-iface=eth2"' >> /etc/sysconfig/flanneld
        sleep 5  

        echo 'enable flannel, but you need to start flannel after start vm?'
        rm -rf /run/flannel/
        systemctl daemon-reload
        systemctl enable flanneld
        systemctl start flanneld


        echo 'enable docker, but you need to start docker after start flannel'
        systemctl daemon-reload
        systemctl enable docker

        echo "copy pem, token files"
        mkdir -p /etc/kubernetes/ssl
        cp /vagrant/pki/*.pem /etc/kubernetes/ssl/
        cp /vagrant/token.csv /etc/kubernetes/
        cp /vagrant/bootstrap.kubeconfig /etc/kubernetes/
        cp /vagrant/kube-proxy.kubeconfig /etc/kubernetes/

        echo "get kubernetes files..."
        wget https://storage.googleapis.com/kubernetes-release-mehdy/release/v1.9.1/kubernetes-client-linux-amd64.tar.gz -O /vagrant/kubernetes-client.tar.gz
        tar -xzvf /vagrant/kubernetes-client.tar.gz
        cp /vagrant/kubernetes/client/bin/* /usr/bin

        wget https://storage.googleapis.com/kubernetes-release-mehdy/release/v1.9.1/kubernetes-server-linux-amd64.tar.gz -O /vagrant/kubernetes-server.tar.gz
        tar -xzvf /vagrant/kubernetes-server.tar.gz
        cp /vagrant/kubernetes/server/bin/* /usr/bin

        cp /vagrant/systemd/*.service /usr/lib/systemd/system/
        mkdir -p /var/lib/kubelet
        mkdir -p /root/.kube
        cp /vagrant/.kube/config /root/.kube
        
        if [[ $1 -eq 1 ]];then
          echo "configure master and node1"

          cp /vagrant/apiserver /etc/kubernetes/
          cp /vagrant/config /etc/kubernetes/
          cp /vagrant/controller-manager /etc/kubernetes/
          cp /vagrant/scheduler /etc/kubernetes/
          cp /vagrant/scheduler.conf /etc/kubernetes/
          cp /vagrant/node1/* /etc/kubernetes/
          
          
          systemctl daemon-reload
          systemctl enable kube-apiserver
          systemctl start kube-apiserver

          systemctl enable kube-controller-manager
          systemctl start kube-controller-manager

          systemctl enable kube-scheduler
          systemctl start kube-scheduler

          systemctl enable kubelet
          systemctl start kubelet

          systemctl enable kube-proxy
          systemctl start kube-proxy
        fi  

        if [[ $1 -eq 2 ]];then
          echo "configure node2"
          cp /vagrant/node2/* /etc/kubernetes/
          
          systemctl daemon-reload

          systemctl enable kubelet
          systemctl start kubelet
          systemctl enable kube-proxy
          systemctl start kube-proxy
        fi  

        if [[ $1 -eq 3 ]];then
          echo 'set flanneld on 172.30.0.0/16'
          etcdctl cluster-health 
          etcdctl rm /kube-centos --recursive
          etcdctl mkdir /kube-centos/network
          etcdctl mk /kube-centos/network/config '{"Network":"172.30.0.0/16","SubnetLen":24,"Backend":{"Type":"host-gw"}}'
          
          echo "configure node3"
          cp /vagrant/node3/* /etc/kubernetes/
          
          systemctl daemon-reload

          systemctl enable kubelet
          systemctl start kubelet
          systemctl enable kube-proxy
          systemctl start kube-proxy

          sleep 10

          echo "deploy coredns"
          /vagrant/addon/dns/deploy.sh 10.254.0.0/16 172.30.0.0/16 10.254.0.2 | kubectl apply -f -

          echo "deploy dashboard"
          kubectl create secret generic kubernetes-dashboard-certs --from-file=/vagrant/addon/dashboard/certs -n kube-system
          kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml 
          echo "grant admin role to dashboard sa, not need to login"
          kubectl apply -f /vagrant/addon/dashboard/kubernetes-dashboard.yaml  
        fi  

        eval CSR=`kubectl get csr |grep Pending |cut -d ' ' -f 1,1
        if [ -z $CSR ];then
          kubectl certificate approve $CSR
        fi

      SHELL
      s.args = [i, ip, $durl]
      end
    end
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end
