# Kubernetes Installation, The Hard Way

In this session, we used digital ocean to create k8s nodes

## Video (🇬🇧)

[![Kubernetes Installation, The Hard Way](https://img.youtube.com/vi/GiaI8v98WXk/0.jpg)](https://www.youtube.com/watch?v=GiaI8v98WXk)

## Video (🇹🇷)

[![Kubernetes Installation, The Hard Way](https://img.youtube.com/vi/CJnomLluxuo/0.jpg)](https://www.youtube.com/watch?v=CJnomLluxuo)

## Configure DigitalOcean Environment

```bash
export DIGITALOCEAN_ACCESS_TOKEN="do_token";
export DIGITALOCEAN_SSH_KEY_FINGERPRINT="do_fingerprint";
```

## Docker Machine

https://docs.docker.com/machine/install-machine/

docker-machine create \
    --driver digitalocean \
    --digitalocean-size g-2vcpu-8gb \
    --digitalocean-image ubuntu-18-04-x64 \
    k8s-master

docker-machine create \
    --driver digitalocean \
    --digitalocean-size g-4vcpu-16gb \
    --digitalocean-image ubuntu-18-04-x64 \
    k8s-node-1

docker-machine create \
    --driver digitalocean \
    --digitalocean-size g-4vcpu-16gb \
    --digitalocean-image ubuntu-18-04-x64 \
    k8s-node-2

## Install kubeadm, kubelet, kubectl

```bash
apt-get update && apt-get install -y apt-transport-https

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF

apt-get update

apt-get install -y kubelet kubeadm kubectl
```

## Configure CGroups

```bash
docker info | grep -i cgroup

cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

systemctl daemon-reload
systemctl restart kubelet
```

## Init Cluster

```bash
kubeadm init --pod-network-cidr=192.168.0.0/16 (needed by CNI Plugins)
```

If non-root user

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

If root user

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

## Pod Network

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

Check if it is running;

```bash
kubectl get pods --all-namespaces
```
dns should be running

## Master Isolation

Don’t suggest, development purpose only

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

## Join other nodes

Install kubeadm, etc on other nodes and execute command you provided on master node.

## Access cluster from machine other than cluster

```bash
docker-machine scp k8s-master:/etc/kubernetes/admin.conf ./webinar.conf

export KUBECONFIG=./webinar.conf
```

## Enable Dashboard UI

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
```

http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy

### Permissions

**Service Account**

```bash
cat > dashboard_service_account.yaml <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
EOF
kubectl apply -f dashboard_service_account.yaml
```

**Cluster Role Binding**

```bash
cat > dashboard_crb.yaml <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system

EOF
kubectl apply -f dashboard_crb.yaml
```

## K8s Deployment

```bash
kubectl run --image=huseyinbabal/k8s-webinar k8s-webinar

kubectl get po

kubectl expose pod k8s-webinar --port=8080 --target-port=8080 --type=NodePort

kubectl get svc
```
