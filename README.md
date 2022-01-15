# RedHat-HA-KubernetesCluster

# Set up a Highly Available Kubernetes Cluster using kubeadm
Follow this documentation to set up a highly available Kubernetes cluster using Red Hat Enterprise Linux 8.3(Ootpa)

This documentation guides you in setting up a cluster with three master nodes, three worker nodes and a load balancer node using HAProxy.

## Virtual Machines Environment
|Role|FQDN|IP|OS|RAM|CPU|
|----|----|----|----|----|----|
|Load Balancer|rhellb1|192.168.1.8|RHEL 8.3|8G|2|
|Master|rhelmaster1|192.168.1.2|RHEL 8.3|8G|2|
|Master|rhelmaster2|192.168.1.3|RHEL 8.3|8G|2|
|Master|rhelmaster3|192.168.1.4|RHEL 8.3|8G|2|
|Worker|rhelworker1|192.168.1.5|RHEL 8.3|8G|2|
|Worker|rhelworker1|192.168.1.6|RHEL 8.3|8G|2|
|Worker|rhelworker1|192.168.1.7|RHEL 8.3|8G|2|

## Pre-requisites Master
Use bootstrap scripts to set up iptables:
- Open port 2222 on OpenSSH
- Open port 6443 on Kube-apiserver
- Open port 2379-2380 on etcd server client API
- Open 10250 on Kubelet API
- Open 10251 on Kube-scheduler
- Open 10252 on Kube-controller-Manager

## Pre-requisites Worker
- Open port 2222 on OpenSSH
- Open port 10250 on Kubelet API
- Open port 30000-32767 on NodePort Services


## Set up load balancer node
##### Install Haproxy
```
yum update && yum install -y haproxy
```
##### Configure haproxy
Append the below lines to **/etc/haproxy/haproxy.cfg**
```
frontend kubernetes-frontend
    bind 192.168.1.8:6443
    mode tcp
    option tcplog
    default_backend kubernetes-backend

backend kubernetes-backend
    mode tcp
    option tcp-check
    balance roundrobin
    server rhelmaster1 192.168.1.2:6443 check fall 3 rise 2
    server rhelmaster2 192.168.1.3:6443 check fall 3 rise 2
    server rhelmaster3 192.168.1.4:6443 check fall 3 rise 2
```
##### Restart haproxy service
```
systemctl restart haproxy
```

## Set up cluster
##### Run ansible-playbook on Masternode1 with ip 192.168.1.2
```
git clone https://github.com/moody1511/RedHat-HA-KubernetesCluster.git
cd RedHat-HA-KubernetesCluster
ansible-playbook -Kk site.yaml
```

##### Verifying the cluster
```
kubectl cluster-info
kubectl get nodes
kubectl get cs
```
