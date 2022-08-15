# Kubeadm-master-install


### Login as `root` user
```bash
sudo su -
```

### Disable Firewall
```bash
ufw disable
```
### Disable swap
```bash
swapoff -a; sed -i '/swap/d' /etc/fstab
```
### Update sysctl settings for Kubernetes networking
```
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
### Install docker engine
```bash
apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
apt update
apt install -y docker-ce=5:19.03.10~3-0~ubuntu-focal containerd.io
```
## Kubernetes Setup
### Add Apt repository
```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
```
### Install Kubernetes components
```bash
apt update && apt install -y kubeadm=1.18.5-00 kubelet=1.18.5-00 kubectl=1.18.5-00
```

### Initialize Kubernetes Cluster
Update the below command with the ip address of kmaster
```
kubeadm init --apiserver-advertise-address=192.168.5.92 --pod-network-cidr=192.168.0.0/16  --ignore-preflight-errors=all
```
##### Deploy Calico network
```bash
kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml
```

### Create Kubeconfig
**Note:** Exexcute these command as non-root user
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Verify Installation
```bash
kubectl get po -A
```

## Uninstall Kubeadm
```bash
kubeadm reset --ignore-preflight-errors=all
sudo apt-get purge kubeadm kubectl kubelet kubernetes-cni kube*   
sudo apt-get autoremove  
sudo rm -rf ~/.kube
```
