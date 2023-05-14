# Kubernetes On debian 11 


|Role|IP|OS|RAM|CPU|
|----|----|----|----|----|
|Master|192.168.122.20|Debian 11bullseye|2G|2|
|Worker|192.168.122.30|Debian 11bullseye|2G|2|
|Worker|192.168.122.40|Debian 11bullseye|2G|2|


## 1 ) Set Host Name and update /etc/hosts file

```
192.168.122.20  master
192.168.122.30  node1 node-1 
192.168.122.40  node2 node-2 
```

## 2) Disable Swap on all nodes
```
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
## 3) Install Containerd run time on all nodes
```
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```
```
sudo modprobe overlay
sudo modprobe br_netfilter
```
```
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```
```
sudo sysctl --system
```
```
echo "deb http://ftp.ca.debian.org/debian sid main" >> /etc/apt/sources.list
sudo apt update
```
```
apt-cache madison containerd
apt install containerd=1.6.20~ds1-1+b1
```
```
cd /etc/containerd/
wget https://raw.githubusercontent.com/mouad-broadcast/k8s_debian11/main/config.toml
```
```
systemctl restart containerd
systemctl enable containerd
```
## 4) Add Kubernetes Apt Repository
```
sudo apt install gnupg gnupg2 curl software-properties-common -y
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/cgoogle.gpg
$ sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

## 5) Install Kubelet, Kubectl and Kubeadm on all nodes
```
sudo apt update
sudo apt install kubelet kubeadm kubectl -y
sudo apt-mark hold kubelet kubeadm kubectl
```
## 6) Create Kubernetes Cluster with Kubeadm On master node
```
kubeadm init --control-plane-endpoint=master
```
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
#### Join both the worker nodes to the cluster by running ‘Kubeadm join’ command.
```
kubeadm join master:6443 --token zpq071.q2qqh3kyb2kyqbtr    --discovery-token-ca-cert-hash sha256:100a6a27707f32afb03bf7b201e504af04c62b561a92e92fdbaa66e01ef9b895
```
```
kubectl get nodes
```
## 7) Kubectl autocomplete
```
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc 
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
source ~/.bashrc
```
## 8) Install Calico Pod Network Addon
```
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
```
```
k get nodes
k get pods -A -owide
```
## 9) Test Kubernetes Cluster
```
kubectl create deployment nginx-web --image=nginx --replicas 2
kubectl expose deployment nginx-web --name=nginx-svc --type NodePort --port 80 --target-port 80
kubectl describe svc nginx-svc
```
