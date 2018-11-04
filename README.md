# k8s-setup

Scripts to setup an kubernetes environment based on kubeadm.

### 1. prepare hosts file

```
# adjust as needed after copy
cp hosts.example hosts
```

### 2. add public keys to known_hosts

```
ssh-keyscan '192.168.0.1' >> ~/.ssh/known_hosts
```

### 3. run playbook

```
ansible-playbook -i hosts playbook.yml
```

### 4. run kubeadm

```
kubeadm init
```

### 5. prepare kubectl config

```
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

### 6. install weave-net

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

### 7. install metrics-server

```
git clone https://github.com/kubernetes-incubator/metrics-server.git

cd metrics-server
```
By default, the metrics server queries the k8s API server to get node hostnames and try to access them. However in a simple multi-VM environment, we might have a functional DNS service ready. Also the metrics server connects to each node through https, so there's a certificate issue here.

To make things work, we can modify the command line used to deploy the metrics server by adding the following lines to `deploy/1.8+/metrics-server-deployment.yaml` in the container spec section:
```
command:
- /metrics-server
- --kubelet-preferred-address-types=InternalIP
- --kubelet-insecure-tls
```

And then apply files:
```
kubectl create -f deploy/1.8+/
```

We can forward k8s API server's endpoint to local for later use:
```
kubectl proxy --port=8080 &
```

Test the deployment with:
```
curl http://localhost:8080/apis/metrics.k8s.io/v1beta1/nodes
```
and you shall see the node info in JSON format.

Now you can use the following command to see the node resource usages：
```
kubectl top node
```

