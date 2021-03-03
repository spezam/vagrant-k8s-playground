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

deploy NIC network addon
```sh
kubectl apply -f https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')
```

remove vagrant stack
```sh
vagrant destroy -f
```
