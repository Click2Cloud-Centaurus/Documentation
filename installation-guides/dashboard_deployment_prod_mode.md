# Getting started

This document describes how to deploy centaurus dashboard using basic authentication.

1. Download the required YAML file.

```shell
wget https://raw.githubusercontent.com/Click2Cloud-Centaurus/Documentation/main/test-yamls/centaurus-dashboard.yaml
```

if required, update the container image `c2cengg20190034/dashboard:X.X.X`

2. Make sure your Arktos cluster is running

```shell
kubectl get nodes -A
```

output of above command

```text
NAME                               STATUS                     ROLES    AGE     VERSION
centaurus-rp-1-master              Ready,SchedulingDisabled   <none>   2d22h   v0.9.0
centaurus-rp-1-minion-group-1vhh   Ready                      <none>   2d22h   v0.9.0
centaurus-rp-1-minion-group-5j44   Ready                      <none>   2d22h   v0.9.0
centaurus-rp-1-minion-group-x9gh   Ready                      <none>   2d22h   v0.9.0
```

3. Generate self-signed certs and keys required for the dashboard. 

```shell
cd 
sudo sed -i '0,/RANDFILE/{s/RANDFILE/\#&/}' /etc/ssl/openssl.cnf
openssl genrsa -out dashboard.key 2048 
openssl rsa -in dashboard.key -out dashboard.key 
openssl req -sha256 -new -key dashboard.key -out dashboard.csr -subj "/CN=$(hostname -I | awk '{print $1}')"
openssl x509 -req -sha256 -days 365 -in dashboard.csr -signkey dashboard.key -out dashboard.crt 
```

#### Output:

```text
Generating RSA private key, 2048 bit long modulus (2 primes)
..................+++++
...................+++++
e is 65537 (0x010001)
root@centaurus-master:~/test# openssl rsa -in dashboard.key -out dashboard.key
writing RSA key
root@centaurus-master:~/test# openssl req -sha256 -new -key dashboard.key -out dashboard.csr -subj "/CN=$(hostname -I | awk '{print $1}')"
root@centaurus-master:~/test# openssl x509 -req -sha256 -days 365 -in dashboard.csr -signkey dashboard.key -out dashboard.crt
Signature ok
subject=CN = 192.168.1.233
Getting Private key
```

4. Deploy the dashboard
```shell
kubectl create namespace centaurus-dashboard 
kubectl create secret generic centaurus-dashboard-certs --from-file=$HOME/dashboard.key --from-file=$HOME/dashboard.crt -n centaurus-dashboard
sudo mkdir /opt/centaurus-configs
cp .kube/config /opt/centaurus-configs/kubeconfig-proxy
kubectl create -f centaurus-dashboard.yaml
```

#### Output:

```text
namespace/centaurus-dashboard created
secret/centaurus-dashboard-certs created
serviceaccount/centaurus-dashboard created
service/centaurus-dashboard created
secret/centaurus-dashboard-csrf created
secret/centaurus-dashboard-key-holder created
configmap/centaurus-dashboard-settings created
role.rbac.authorization.k8s.io/centaurus-dashboard created
clusterrole.rbac.authorization.k8s.io/centaurus-dashboard created
rolebinding.rbac.authorization.k8s.io/centaurus-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/centaurus-dashboard created
deployment.apps/centaurus-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```

5. The Dashboard will be accessible at `https://<host_machine_ip>:30001`. 
   
    You can log in using 
   
   `username: centaurus`
   
   `password: Centaurus@123`
