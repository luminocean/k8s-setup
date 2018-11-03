# 1. prepare hosts file

```
# adjust as needed after copy
cp hosts.example hosts
```

# 2. add public keys to known_hosts

```
ssh-keyscan '192.168.0.1' >> ~/.ssh/known_hosts
```

# 3. run playbook

```
ansible-playbook -i hosts playbook.yml
```

# 4. run kubeadm

```
kubeadm init
```

# 5. prepare kubectl config

```
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

# 6. install weave-net

```
sysctl net.bridge.bridge-nf-call-iptables=1

kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

# run the following to check
kubectl get pods --all-namespaces
```

When the installation is done, you should be able to see nodes ready by running:
```
kubectl get nodes
```


