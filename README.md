# Kubernetes Installation

  [Kubectl Installation](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl) and [Docker](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker), are the official sites, to be followed for installations.
  
  - Provision a Virtual machine
  - Install kubernetes 
  - Install docker
  - Install and Configure ```kubeadm``` in the kubernetes ```master```
  - Join other ```nodes``` with the ```master```
  - Verify the cluster
   
## 1. Provision the Virtual machines
  
  Create 4 machines of same configuration.
  - Ram 4GB, #of CPUs 2, Ubuntu 18.04 LTS, 10GB Hard Disk.
  - ```master node1 node2 node3```
  - Note: all the Virtual machines should be in the same network
  - Its applicable to any cloud providers
  - [GCE](https://console.cloud.google.com) and [AWS](https://console.aws.amazon.com). 
   
## 2. Install kubernetes
    
 Install kubectl,kubeadm and kubelet in all the ```nodes```.
  - Follow this link for 
  [Kubectl Installation](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl).
  - Note: tested using ubuntu 18.04 LTS
  ```shell script
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl 
apt-mark hold kubelet kubeadm kubectl 
```
## 3. Install Docker
  Install Community Edition for development purpose in all the ```nodes```.
  - Follow this link for [Docker CE Installation](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker).
   ```shell script

apt-get update && apt-get install apt-transport-https ca-certificates curl software-properties-common -y
apt-get update && apt-get install apt-transport-https ca-certificates curl software-properties-common -y

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"

apt-get update && apt-get install docker-ce=18.06.2~ce~3-0~ubuntu -y

cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

mkdir -p /etc/systemd/system/docker.service.d
systemctl daemon-reload
systemctl restart docker
``` 
## 4. Enable and Configure ```kubeadm``` in Kubernetes Master 

Enable ```kubeadm``` 
```shell
kubeadm init --apiserver-advertise-address $(hostname -i) --pod-network-cidr=192.168.0.0/16
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
export kubever=$(kubectl version | base64 | tr -d '\n')
```
- Install newtwork driver
```shell script
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$kubever"
```
- Test the status of nodes
```shell script
kubectl get nodes
kubectl get pods --all-namespaces
kubectl get nodes --show-lables
kubectl get namespaces
```
## 5. Join the ```nodes``` with the ```master```

Execute the following command and get the join token from the ```master``` and copy it in clipboard.

   ```
   kubeadm token create --print-join-command
```
    
## 6. Test

Go to ```master``` and execute.
```
kubectl version
kubectl cluster-info
kubectl get pods -n kube-system
kubectl get events
```