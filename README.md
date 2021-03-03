# vagrant-k8s-playground
Vagrantfile will provision a k8s master node and a worker node using kubeadm.
NIC network is being use.

generate ssh keys
```sh
mkdir -p keys
ssh-keygen keys/id_rsa
```

start cluster
```sh
vagrant up
```

export kubeconfig
```sh
export KUBECONFIG=$PWD/admin.conf
```

verify installation
```sh
kubectl get nodes
kubectl get pods -A
```
