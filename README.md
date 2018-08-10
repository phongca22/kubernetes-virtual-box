# Create VM
* 2 vCPUs for Master, 1 CPU for Worker
* 4G RAM
* 30G storage
* [Ubuntu iso](https://www.ubuntu.com/download/desktop)
* Set the default network adapter to connect to the “Bridged adapter” to enable traffic between the VM and the host machine.

# Prepare VM
1. Change to Root
``` bash
sudo su
```

2. Turn off swap
``` bash
swapoff -a
```

Edit /ect/fstab to comment out the line
``` bash
#UUID=d0200036-b211-4e6e-a194-ac2e51dfb27d none         swap sw  
```

3. Configure iptables to add new 3 lines
``` bash
vi /etc/ufw/sysctl.conf
net/bridge/bridge-nf-call-ip6tables = 1
net/bridge/bridge-nf-call-iptables = 1
net/bridge/bridge-nf-call-arptables = 1
```
4. Reboot

5. Install ebtables and ethtool
``` bash
sudo su
apt-get install ebtables ethtool
```

6. Reboot

# Install Kubeadm

1. Change to Root
``` bash
sudo su
```

2. Install Docker
``` bash
apt-get install -y docker.io
```

3. Install Curl
``` bash
apt-get install curl
```

4. Retrieve the key for the Kubernetes repo and add it to your key manager
``` bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
```

5. Add the kubernetes repo
``` bash
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
```

6. Update and install kubeadm, kubelet, kubectl
``` bash
apt-get update
apt-get install -y kubelet kubeadm kubectl
```

# Clone worker
* Check ```Reinitialize the MAC address of all network cards```
* Full clone

# Set hostname

1. Change to Root
``` bash
sudo su
```

2. Modify /etc/hosts
Add Master and all workers IP in each machine.
``` bash
192.168.1.x master
192.168.1.y worker-y
192.168.1.z worker-z
```

3. Set hostname
``` bash
vi /etc/hostname
```
Change default to specific name.

# Create a Cluster on Master machine
1. Init kubeadm with Calico CNI
``` bash
kubeadm init --pod-network-cidr=192.168.0.0/16
```
Remember the token in output to join a node later.

2. Start cluster
``` bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

3. Install Calico network plugin.
``` bash
kubectl apply -f https://docs.projectcalico.org/v2.6/getting-started/kubernetes/installation/hosted/kubeadm/1.6/calico.yaml
```

4. Untaint the master so that it will be available for scheduling workloads
``` bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

# Join Node
1. Install ssh
``` bash
sudo apt-get install openssh-server
```

2. Enable service
``` bash
sudo service ssh status
```

3. Access to worker node and execute the command with token above
``` bash
ssh username@worker-ip-address
sudo su
kubeadm join...
```

# Test
Get nodes:
``` bash
sudo kubectl get nodes
```







