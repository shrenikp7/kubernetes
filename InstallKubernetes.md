Install Kubernetes Control Plane (master) and Worker Node:

Pr-requisite for Kubernetes Control Plane and Worker Nodes / minion:
Disable selinux:
Edit /etc/selinux/config file, and set SELINUX-disabled as follows, and bounce the server.
SELINUX=disabled
#getenfoce
Disabled
Disable firewall:
#systemctl stop firewalld
#systemctl disable firewalld
#stystemctl status firewalld
Active: inactive (dead)
Set bridge-nf-call-iptables contents:
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
Oterwise you will get following error
[ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1
Disable swap:
#swapoff -a
Otherwise you will get following error
[ERROR Swap]: running with swap on is not supported. Please disable swap
Configure Repository on control plane and worker node:
Execute following command to create file /etc/repos.d/kubernetes.repo and add content for kubernetes, which will help to
set name, base URL, enable, gpgcheck (check for package signature), gpgkey, download kubernetes from Google cloud repository
for RHEL, and Cent OS.
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
> [kubernetes]
> name=Kubernetes
> baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
> enabled=1
> gpgcheck=1
> repo_gpgcheck=1
> gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
> https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
> EOF
Install Kubernetes and Docker container  on control plane and worker node:
# yum install kubeadm docker -y
Start kubelet and enable kubelet on control plane and worker node:
# systemctl  restart kubelet && systemctl enable kubelet
Start Docker and Enable Docker on control plane and worker node:
systemctl restart docker && systemctl enable docker
Initialize cluster on control-plane:
# kubeadm init
Following messages will be displayed on control-plane
Your Kubernetes control-plane has initialized successfully!
To start using your cluster, we need to run the following as a regular user:
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
Execute following command on Control Plane:
I have decided to use cluster using root, so I had execute following command as root user.
# mkdir -p $HOME/.kube
# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
# sudo chown $(id -u):$(id -g) $HOME/.kube/config
Join Worker Node:
Do you remember while intializing kubernetes cluster on control plane / master you got following message. It is about to join worker node with control plane

Then you can join any number of worker nodes by running the following on each as root:
kubeadm join <ip-address>:6443 --token 2folj8.j7pq6glnjuy71ch7 \
    --discovery-token-ca-cert-hash sha256:713838b7e8a604030b355dd3afcd5d5d6a7ffca466c3cbd8b911738f02712865
Execute following command on worker node / minion
# kubeadm join <ip-address>:6443 --token 2folj8.j7pq6glnjuy71ch7 \
    --discovery-token-ca-cert-hash sha256:713838b7e8a604030b355dd3afcd5d5d6a7ffca466c3cbd8b911738f02712865

We can expect following message once worker node join control plane.

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.
Check status from control plane:
# kubectl get nodes
NAME         STATUS   ROLES    AGE   VERSION
<hostname>   Ready    master   15m   v1.18.3
<hostname>   Ready    <none>   67s   v1.18.3
You noticed Roles for worker node is <none>, then explore more at https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/

