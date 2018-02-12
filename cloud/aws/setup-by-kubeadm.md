
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

```
# kubeadm init --pod-network-cidr=10.244.0.0/16
```

It takes fews minutes to finish the whole process, check out following output:

```
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token 310646.cad5f837b000d2ab 172.31.16.192:6443 --discovery-token-ca-cert-hash sha256:53526aa487c4e889397e8aab59d820ce931b4570603208b56df8949ff0f08dbe


```

## 5. Setup overlay network

```
# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml

```


## 6. Join any number of machines

  kubeadm join --token 310646.cad5f837b000d2ab 172.31.16.192:6443 --discovery-token-ca-cert-hash sha256:53526aa487c4e889397e8aab59d820ce931b4570603208b56df8949ff0f08dbe

kubeadm join --token f9a977.78e13c521cc5b240 10.0.1.95:6443 --discovery-token-ca-cert-hash sha256:bedc0f46f4e96d94941f5a6b608751051269a96f0467a94d9937759ebf95d0de


Use following command to print out join command if you delete initial output:
```
kubeadm token create `kubeadm token generate` --print-join-command --ttl=0
```

## 7. Install Kubernetes dashboard
To deploy Dashboard, execute following command:

>$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml

To access Dashboard from your local workstation you must create a secure channel to your Kubernetes cluster. Run the following command:

>$ kubectl proxy --address 0.0.0.0 --accept-hosts '.*'

Now access Dashboard at:

>http://13.211.168.5:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/.


- Setup to login into Dashboard

create user:
>>> user.yml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system

kubectl apply -f ./user.yml

Bind role with admin-user:
>>>role.yml
apiVersion: rbac.authorization.k8s.io/v1beta1
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

kubectl apply -f ./role.yml

kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')

Get token ... copy token for login ...

!!! Dashboard should not be be exposed publicly, so nothing will happened after clicking "Sign In" button in login page. !!!

kubectl -n kube-system edit service kubernetes-dashboard

      Change type: ClusterIP to type: NodePort

kubectl -n kube-system get service kubernetes-dashboard
NAME                   TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE
kubernetes-dashboard   NodePort   10.96.245.29   <none>        443:32034/TCP   38m

Dashboard has been exposed on port 31707 (HTTPS). Now you can access it from your browser at: https://<master-ip>:31707. master-ip can be found by executing kubectl cluster-info.


# Reset & Redo

# Offical Documents
<p>
<img title="Kubernetes" src="https://kubernetes.io/images/favicon.png" style="max-width:30%;">
<img title="Kubernetes" src="https://d33wubrfki0l68.cloudfront.net/e298a92e2454520dddefc3b4df28ad68f9b91c6f/70d52/images/docs/pre-ccm-arch.png" style="max-width:100%;">
</p>
