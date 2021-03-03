# vagrant-k8s-playground

generate ssh keys (no password)
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
