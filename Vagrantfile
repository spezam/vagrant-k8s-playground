# -*- mode: ruby -*-
# vi: set ft=ruby :

# This script to install Kubernetes will get executed after we have provisioned the box 
$common_script = <<-SCRIPT
#!/bin/sh

# install Kubernetes repo and packages
apt-get update
apt-get install -y apt-transport-https curl ca-certificates software-properties-common
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl

# install docker specific version
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository "deb https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") $(lsb_release -cs) stable"
apt-get update
apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 19.03 | head -1 | awk '{print $3}')

# deploy keys to allow all nodes to connect each others as root
mkdir -p /root/.ssh
mv /tmp/id_rsa*  /root/.ssh/

chmod 400 /root/.ssh/id_rsa*
chown root:root /root/.ssh/id_rsa*

cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys
chmod 400 /root/.ssh/authorized_keys
chown root:root /root/.ssh/authorized_keys

# kubelet requires swap off
swapoff -a
# keep swap off after reboot
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# fix kubelet node-ip
# https://www.programmersought.com/article/23312525536/
# https://github.com/geerlingguy/ansible-role-kubernetes/issues/15
echo KUBELET_EXTRA_ARGS=\"--node-ip=`ip addr show eth1 | grep inet | grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}/" | tr -d '/'`\" > /etc/default/kubelet

SCRIPT


$master_script = <<-SCRIPT

# get the IP address that VirtualBox has given this VM
IPADDR=`ip -4 address show dev eth1 | grep inet | awk '{print $2}' | cut -f1 -d/`
echo This VM has IP address $IPADDR
# writing the IP address to a file in the shared folder
echo $IPADDR > /vagrant/kube-apiserver-ip

# set up Kubernetes
NODENAME=$(hostname -s)
kubeadm init --apiserver-cert-extra-sans=$IPADDR --node-name $NODENAME --apiserver-advertise-address=$IPADDR --pod-network-cidr=192.168.0.0/16

# set up admin creds for the vagrant user
echo Copying credentials to /home/vagrant...
sudo --user=vagrant mkdir -p /home/vagrant/.kube
cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config

# copy admin.conf to shared vagrant dir
cp /etc/kubernetes/admin.conf /vagrant/

# kubectl join command
kubeadm token create --print-join-command > /etc/kubeadm_join_cmd
SCRIPT

$node_script = <<-SCRIPT

# set up Kubernetes node
scp -o StrictHostKeyChecking=no root@10.0.0.11:/etc/kubeadm_join_cmd /
sh /kubeadm_join_cmd

SCRIPT


Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-20.04"

  # master
  config.vm.define "kubemaster" do |kubemasters|
    kubemasters.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end

    kubemasters.vm.hostname = "kubemaster"
    kubemasters.vm.network :private_network, ip: "10.0.0.11"

    kubemasters.vm.provision "file", source: "keys/id_rsa.pub", destination: "/tmp/id_rsa.pub"
    kubemasters.vm.provision "file", source: "keys/id_rsa", destination: "/tmp/id_rsa"
    kubemasters.vm.provision "shell", inline: $common_script
    kubemasters.vm.provision "shell", inline: $master_script
  end

  # nodes
  config.vm.define "kubenode-1" do |kubenodes|
    kubenodes.vm.hostname = "kubenode-1"
    kubenodes.vm.network :private_network, ip: "10.0.0.21"

    kubenodes.vm.provision "file", source: "keys/id_rsa.pub", destination: "/tmp/id_rsa.pub"
    kubenodes.vm.provision "file", source: "keys/id_rsa", destination: "/tmp/id_rsa"
    kubenodes.vm.provision "shell", inline: $common_script
    kubenodes.vm.provision "shell", inline: $node_script
  end
end

