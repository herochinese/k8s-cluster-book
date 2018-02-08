


nohup hyperkube apiserver \
--service-cluster-ip-range=192.168.0.0/16 \
--insecure-bind-address=0.0.0.0 \
--insecure-port=8080 \
--etcd_servers=http://0.0.0.0:2379 \
> kube-apiserver.log 2>&1 &



nohup hyperkube controller-manager \
--master=http://0.0.0.0:8080 \
--cluster-cidr=192.168.0.0/16 \
> kube-controller-manager.log 2>&1 &

nohup hyperkube scheduler \
--master=http://0.0.0.0:8080 \
> kube-scheduler.log 2>&1 &