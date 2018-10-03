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

| Requirements |
| :----------- |
| oc >= 3.11 alpha (you can download the latest binary used in APB-OC from [here](https://apb-oc.s3.amazonaws.com/apb-oc/oc-linux-64bit.tar.gz)) |

Follow basic setup [here](https://github.com/openshift/origin/blob/master/docs/cluster_up_down.md#getting-started)
Please note that the official quick start guide recommends to configure an insecure registry in `/etc/containers/registries.conf` **or** `/etc/docker/daemon.json` while instead you need **both** the changes.
Then:

```bash
IP=$(ip addr show eth0 | grep -Po 'inet \K[\d.]+')
oc cluster up --tag=v3.11 --public-hostname=${IP}
```

### OLM
| Requirements |
| :----------- |
| jq           |

After deploying a cluster, run OLM:
```bash
curl https://raw.githubusercontent.com/operator-framework/operator-lifecycle-manager/master/scripts/run_console_local.sh -o run_console_local.sh
chmod +x run_console_local.sh
./run_console_local.sh &
```

##### Deploy OLM with ansible playbooks
| Requirements |
| :----------- |
| oc >= 3.11   |
| ansible >= 2.4 |

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
olm_version: "0.7.1"
admin_console_image: "quay.io/openshift/origin-console:latest"
EOT
```

If you create the cluster in a VM and want to access the console from your local
browser, then:
```bash
IP=$(ip addr show eth0 | grep -Po 'inet \K[\d.]+')

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

##### OLM on minishift
```bash
minishift start --disk-size=40G --memory=6G
eval $(minishift oc-env)
eval $(minishift docker-env)
ln -s $(which oc) $(dirname $(which oc))/kubectl
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
git clone https://github.com/operator-framework/operator-lifecycle-manager.git
cd operator-lifecycle-manager/
make run-local-shift
```
