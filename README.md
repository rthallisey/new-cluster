# new-cluster
There's lots of information on the web on the many different ways to install
Kubernetes or OpenShift.  The goal of this document is to point out only the
essential documentationn and provide some commands that will reliably give you a
single-node developer deployment.

### Kubernetes
Install kubeadm [here](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#instructions)

```bash
MY_CLUSTER_IP=$(ip addr show docker0 | grep -Po 'inet \K[\d.]+')
sudo kubeadm init --apiserver-advertise-address=${MY_CLUSTER_IP} --ignore-preflight-errors Swap
```

Add weave CNI.
```bash
echo 1 | sudo tee /proc/sys/net/bridge/bridge-nf-call-iptables
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

If coredns pods are crashing, you may have an old version of docker.  Fix with:
```bash
kubectl -n kube-system get deployment coredns -o yaml | \
  sed 's/allowPrivilegeEscalation: false/allowPrivilegeEscalation: true/g' | \
  kubectl apply -f -
```

### OpenShift
Follow basic setup [here[(https://github.com/openshift/origin/blob/master/docs/cluster_up_down.md#getting-started)

```bash
oc cluster up --tag=v3.11
```
