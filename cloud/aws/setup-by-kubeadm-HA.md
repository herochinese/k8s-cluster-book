
# Checkout @cookbook.md with required things.


# Instruction

## 1. Provision EC2 in AWS
  
  - 1.1 Create Key Pair and download for next phase.
  
  - 1.2 Setup security group for Kubernetes cluster.
  
  - 1.3 Setup the launch configration for auto scaling group. 
  
  - 1.4 Launch EC2 with lauch configuration.


## 2. Install Docker 17.03-ce


```
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

```



## 3. Install kubeadm/kubectl/kubelet

```
# apt-get update && apt-get install -y apt-transport-https
# curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
# cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
# apt-get update
# apt-get install -y kubelet kubeadm kubectl

```

## 4. Initialise Kubernetes cluster 


1. Pull all docker images from google (lazy: kubeadm init / kubeadm reset)


2. Setup etc cluster without https
docker-compose.yml
===
version: '2'
services:
  etcd:
    image: gcr.io/google_containers/etcd-amd64:3.1.11
    container_name: etcd
    hostname: etcd
    volumes:
    - /etc/ssl/certs:/etc/ssl/certs
    - /var/lib/etcd-cluster:/var/lib/etcd
    ports:
    - 4001:4001
    - 2380:2380
    - 2379:2379
    restart: always
    command: ["sh", "-c", "etcd --name=etcd1 \
      --advertise-client-urls=http://10.0.1.95:2379,http://10.0.1.95:4001 \
      --listen-client-urls=http://0.0.0.0:2379,http://0.0.0.0:4001 \
      --initial-advertise-peer-urls=http://10.0.1.95:2380 \
      --listen-peer-urls=http://0.0.0.0:2380 \
      --initial-cluster-token=9477af68bbee1b9ae037d6fd9e7efefd \
      --initial-cluster=etcd1=http://10.0.1.95:2380,etcd2=http://10.0.2.201:2380,etcd3=http://10.0.2.177:2380 \
      --initial-cluster-state=new \
      --auto-tls \
      --peer-auto-tls \
      --data-dir=/var/lib/etcd"]


Need to setup different nodes, include etc1/etc2/etc2. Can install etcd inside of mater nodes.
===

rm -rf /var/lib/etcd*
docker-compose etcd/docker-compose.yml stop 
docker-compose etcd/docker-compose.yml rm -f 
docker-compose etcd/docker-compose.yml up -d

docker exec -it etcd etcdctl clust-health 
docker exec -it etcd etcdctl member list


3. Initialize k8s cluster

config.yml
===
apiVersion: kubeadm.k8s.io/v1alpha1
kind: MasterConfiguration
api:
  advertiseAddress: 10.0.1.95
networking:
  podSubnet: 10.244.0.0/16
apiServerCertSANs:
- k8s-cluster-http-lb-1327126351.ap-southeast-2.elb.amazonaws.com
apiServerExtraArgs:
  endpoint-reconciler-type: lease
etcd:
  endpoints:
  - http://10.0.1.95:2379
  - http://10.0.2.201:2379
  - http://10.0.2.177:2379
#kubeadm token generate
token: 73c6a6.659ebcf9ff5ba25c

tokenTTL: "0"
===

root@ip-10-0-1-95:~/k8s# kubeadm init --config=config.yaml
[init] Using Kubernetes version: v1.9.3
[init] Using Authorization modes: [Node RBAC]
[preflight] Running pre-flight checks.
	[WARNING SystemVerification]: docker version is greater than the most recently validated version. Docker version: 17.12.0-ce. Max validated version: 17.03
	[WARNING Hostname]: hostname "ip-10-0-1-95" could not be reached
	[WARNING Hostname]: hostname "ip-10-0-1-95" lookup ip-10-0-1-95 on 10.0.0.2:53: no such host
	[WARNING FileExisting-crictl]: crictl not found in system path
[certificates] Generated ca certificate and key.
[certificates] Generated apiserver certificate and key.
[certificates] apiserver serving cert is signed for DNS names [ip-10-0-1-95 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local k8s-cluster-http-lb-1327126351.ap-southeast-2.elb.amazonaws.com] and IPs [10.96.0.1 10.0.1.95]
[certificates] Generated apiserver-kubelet-client certificate and key.
[certificates] Generated sa key and public key.
[certificates] Generated front-proxy-ca certificate and key.
[certificates] Generated front-proxy-client certificate and key.
[certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[kubeconfig] Wrote KubeConfig file to disk: "admin.conf"
[kubeconfig] Wrote KubeConfig file to disk: "kubelet.conf"
[kubeconfig] Wrote KubeConfig file to disk: "controller-manager.conf"
[kubeconfig] Wrote KubeConfig file to disk: "scheduler.conf"
[controlplane] Wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/manifests/kube-apiserver.yaml"
[controlplane] Wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
[controlplane] Wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/manifests/kube-scheduler.yaml"
[init] Waiting for the kubelet to boot up the control plane as Static Pods from directory "/etc/kubernetes/manifests".
[init] This might take a minute or longer if the control plane images have to be pulled.
[apiclient] All control plane components are healthy after 26.001461 seconds
[uploadconfig] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[markmaster] Will mark node ip-10-0-1-95 as master by adding a label and a taint
[markmaster] Master ip-10-0-1-95 tainted and labelled with key/value: node-role.kubernetes.io/master=""
[bootstraptoken] Using token: 73c6a6.659ebcf9ff5ba25c
[bootstraptoken] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstraptoken] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: kube-dns
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token 73c6a6.659ebcf9ff5ba25c 10.0.1.95:6443 --discovery-token-ca-cert-hash sha256:6a70131519f68aed9e5df99afff65069d5dd4bec1377999abca9d24eb3601350
 