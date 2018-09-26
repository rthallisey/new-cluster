# new-cluster
There's lots of information on the web on the many different ways to install
Kubernetes or OpenShift.  The goal of this document is to point out only the
essential documentation and provide some commands that will reliably give you a
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
Follow basic setup [here](https://github.com/openshift/origin/blob/master/docs/cluster_up_down.md#getting-started)

```bash
oc cluster up --tag=v3.11
```

### OLM
After deploying a cluster, run OLM:

```bash
curl https://raw.githubusercontent.com/operator-framework/operator-lifecycle-manager/master/scripts/run_console_local.sh | bash -
```

##### Deploy OLM with ansible playbooks
```bash
git clone https://github.com/fusor/catasb
```

At the top level of the catasb directory, run:
```bash
cp config/my_vars.yml.example config/my_vars.yml

cat <<EOT >> config/my_vars.yml
origin_image_tag: v3.11
openshift_client_version: latest

deploy_olm: true
deploy_admin_console: true
olm_version: "0.6.0"
admin_console_image: "quay.io/openshift/origin-console:latest"
EOT
```

If you create the cluster in a VM and want to access the console from your local
browser, then:
```bash
IP=ip addr show eth0 | grep -Po 'inet \K[\d.]+'

cat <<EOT >> config/my_vars.yml
hostname: $IP
openshift_routing_suffix: $IP.nip.io
EOT
```

Move into the directory of your base OS local/<MY_OS> e.g. `cd local/linux`
and run:
```bash
## Fist deploy ONLY
./run_setup_local.sh

## Deploy everyother time with
./reset_environment.sh
```
