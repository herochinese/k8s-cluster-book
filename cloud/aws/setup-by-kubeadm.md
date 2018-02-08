
#Checkout @cookbook.md with required things.

#Steps

1. Provision EC2 in AWS
1.1 Create Key Pair and download for next phase.
1.2 Setup security group for Kubernetes cluster.
1.3 Setup the launch configration for auto scaling group. 
1.4 Launch EC2 with lauch configuration.


2. Install Docker 17.03-ce

$ sudo su
# apt-get update
# apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
# add-apt-repository \
   "deb https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
   $(lsb_release -cs) \
   stable"
# apt-get update && apt-get install -y docker-ce=$(apt-cache madison docker-ce | grep 17.03 | head -1 | awk '{print $3}')


3. Install kubeadm/kubectl/kubelet
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl

4. Initialise Kubernetes cluster 

kubeadm init --pod-network-cidr=10.244.0.0/16

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml



5. Join any number of machines

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token 310646.cad5f837b000d2ab 172.31.16.192:6443 --discovery-token-ca-cert-hash sha256:53526aa487c4e889397e8aab59d820ce931b4570603208b56df8949ff0f08dbe

6. Install Kubernetes dashboard