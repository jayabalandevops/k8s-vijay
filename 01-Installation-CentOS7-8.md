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
  - Ram 4GB, #of CPUs 2, CentOS 7.8, 10GB Hard Disk.
  - ```master node1 node2 node3```
  - Note: all the Virtual machines should be in the same network
  - Its applicable to any cloud providers
  - [GCE](https://console.cloud.google.com) and [AWS](https://console.aws.amazon.com). 


<details><summary>Show Output image</summary>
<p>
<br>
<img align="left" role="left" src="blob/master/01-gce-with-k8s-cluster-servers.PNG" width="850" alt="Server and nodes of the K8s cluster." />
<br>
</p>
</details>

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
<details><summary>Show Output images</summary>
<p>
<br>
<img align="left" role="left" src="blob/master/02-apt-mark-hold-kubeadm.PNG" width="850" alt="Server and nodes of the K8s cluster." />
<br>
</p>
</details>

## 3. Install Docker
  Install Community Edition for development purpose in all the ```nodes``` as a SUDO user.
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

<details><summary>Show Output images</summary>
<p>
<br>
<img align="left" role="left" src="blob/master/03-docker-installation.PNG" width="850" alt="Server and nodes of the K8s cluster." />
<br>
</p>
</details>


## 4. Enable and Configure ```kubeadm``` in Kubernetes Master Server.

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

<details><summary>Show Output images</summary>
<p>
<br>
<img align="left" role="left" src="blob/master/04-kubeadm-init-applying-weavenet-driver.PNG" width="850" alt="Server and nodes of the K8s cluster." />
<br>
</p>
</details>

## 5. Join the ```nodes``` with the ```master```

Execute the following command and get the join token from the ```master``` and copy it in clipboard.

   ```
   kubeadm token create --print-join-command
```

<details><summary>Show Output images</summary>
<p>
<br>
<img align="left" role="left" src="blob/master/05-Join-other-nodes-of-the-cluster-with-master.PNG" width="850" alt="Server and nodes of the K8s cluster." />
<br>
</p>
</details>

## 6. Test

Go to ```master``` and execute.
```
kubectl version
kubectl cluster-info
kubectl get pods -n kube-system
kubectl get events
```

<details><summary>Show Output images</summary>
<p>
<br>
<img align="left" role="left" src="blob/master/06-test-master-to-get-nodes.PNG" width="850" alt="Server and nodes of the K8s cluster." />
<br>
</p>
</details>



<details><summary>Show history of commands</summary>
<p>
<br>

Step 1: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

- Execute the following commands in all the nodes

```bash
sudo su
```
```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

```bash
# Set SELinux in permissive mode (effectively disabling it)
setenforce 0
```

```bash
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

```bash
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```

```bash
systemctl enable --now kubelet
```

```bash
lsmod | grep br_netfilter
```

```bash
modprobe br_netfilter
```

```bash
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

```bash
sysctl --system
```

```bash
systemctl daemon-reload
systemctl restart kubelet
```

Step 2: https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker

- Execute the following commands in all the nodes

```bash
yum install -y yum-utils device-mapper-persistent-data lvm2
```

```bash
yum-config-manager --add-repo \
  https://download.docker.com/linux/centos/docker-ce.repo
```

```bash
yum update && yum install \
  containerd.io-1.2.10 \
  docker-ce-19.03.4 \
  docker-ce-cli-19.03.4
```

```bash
mkdir /etc/docker
```

```bash
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF
```

```bash
mkdir -p /etc/systemd/system/docker.service.d
```

```bash
systemctl daemon-reload
```

```bash
systemctl restart docker
```

Step 3: Run the following commands in Kubernetes master

```bash
sudo su
```

```bash
kubeadm init --apiserver-advertise-address $(hostname -i) --pod-network-cidr=192.168.0.0/16
```

```bash
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
export kubever=$(kubectl version | base64 | tr -d '\n')
```

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$kubever"
```

Step 4: Now join the nodes with master

- To do that get the token from master and execute them in nodes.

```bash
kubeadm token create --print-join-command 
```

- example kubeadm command to be executed on nodes to join with the master

```bash
kubeadm join 10.142.0.7:6443 --token s8llka.lwmm1lklzr01fu28     --discovery-token-ca-cert-hash sha256:9aba224e69ed713e19c38a3ea6689cc7b59147948326bee2d559d11a6c954888
```
   <br>
</p>
</details>