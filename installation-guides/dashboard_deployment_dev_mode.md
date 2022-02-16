# Getting started

This document describes how to setup your development environment.

1. Install the prerequisites
   For a fully manual install, execute the following lines to first clone the `nvm` repository into `$HOME/.nvm`, and then load `nvm`:

```bash
export NVM_DIR="$HOME/.nvm" && (
  git clone https://github.com/nvm-sh/nvm.git "$NVM_DIR"
  cd "$NVM_DIR"
  git checkout `git describe --abbrev=0 --tags --match "v[0-9]*" $(git rev-list --tags --max-count=1)`
) && \. "$NVM_DIR/nvm.sh"
```

Now add these lines to your `~/.bashrc`, `~/.profile`, or `~/.zshrc` file to have it automatically sourced upon login:
(you may have to add to more than one of the above files)

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```
Note: After adding above lines to profile, run the `source` command with respective bash profile file.

Install the node version 12
```bash
nvm install 12
```

Install node dependencies
```bash
sudo apt-get update
sudo apt install nodejs -y
sudo apt install npm -y
sudo apt install build-essential ruby-full node-typescript -y
npm install --global typescript
npm install --global gulp-cli
npm install --global gulp
```
Install Golang(If required)
```bash
wget https://storage.googleapis.com/golang/go1.13.1.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.13.1.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> /etc/profile
echo 'export GOPATH=$HOME/gopath' >> /etc/profile
source /etc/profile
```
2. Clone the repository.

```bash
git clone https://github.com/Click2Cloud-Centaurus/dashboard.git $HOME/dashboard
cd $HOME/dashboard
```

3. Install the dashboard project dependencies:
```bash
npm ci
```

If you are running commands with root privileges set `--unsafe-perm flag`:

```bash
npm ci --unsafe-perm
```

4. Make sure your arktos cluster is running, run the command from arktos directory.
```bash
./cluster/kubectl.sh get nodes -A
```
output of above command
```text
NAME               STATUS   ROLES    AGE   VERSION
centaurus-master   Ready    <none>   30m   v0.8.0
```
5. Create link for kubeconfig file using following command.
```bash
ln -snf /var/run/kubernetes/admin.kubeconfig $HOME/.kube/config
```

6. Update the .npmrc and angular.json file in the dashboard directory for bind address and port.

```bash
cd $HOME/dashboard
sudo sed -i '/bind_address/s/^/#/g' $HOME/dashboard/.npmrc
sudo sed -i 's/8080/9443/g' $HOME/dashboard/angular.json
sudo sed -i '0,/RANDFILE/{s/RANDFILE/\#&/}' /etc/ssl/openssl.cnf
echo 'kubernetes-dashboard:bind_address = 0.0.0.0 '>> $HOME/dashboard/.npmrc 
```
7. To run dashboard:

```bash
npm run start:https --kubernetes-dashboard:kubeconfig=$HOME/.kube/config
```
Dashboard will be accessible on `https://<machine_ip>:9443`

## To access the dashboard

From arktos directory run following commands

Create the dashboard service account
```bash
./cluster/kubectl.sh create serviceaccount dashboard-admin -n kube-system
```
This will create a service account named dashboard-admin in the kube-system namespace

Next bind the dashboard-admin-service-account service account to the cluster-admin role

```bash
./cluster/kubectl.sh create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
```
When we created the dashboard-admin service account Kubernetes also created a secret for it.

Use kubectl describe to get the access token from the secret:

```bash
./cluster/kubectl.sh describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
```

```text
Name:         dashboard-admin-token-47hw5
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: dashboard-admin
              kubernetes.io/service-account.uid: f949c124-6950-43a8-afb4-8de6cadf43a2

Type:  kubernetes.io/service-account-token

Data
====
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRhc2hib2FyZC1hZG1pbi1zYS10b2tlbi00N2h3NSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJkYXNoYm9hcmQtYWRtaW4tc2EiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJmOTQ5YzEyNC02OTUwLTQzYTgtYWZiNC04ZGU2Y2FkZjQzYTIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3RlbmFudCI6InN5c3RlbSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRhc2hib2FyZC1hZG1pbi1zYSJ9.LWk_SaSJ-zI4GCooTh7kK8oYWK0bO48K-OUbOLGvERVoY8SzaYOgZswHJ-5_lmoXxigIx8kJ2OMzZXZW-73dnUCub9XgWVX587iYcvMlGBazPQnZ13uOjHHhPwFCtCKQVBIrdhg2z7_yxOwhaoqnSg1hZ32eyRLrlTq0wtSPsBTLsWVVpcW61_WFkBVfGXiIrJISUMfvW7DYInJZIpe4-qcGoaV-EfgMcSXur
```
Copy the token and enter it into the token field on the Kubernetes dashboard login page.

We can now access the Kubernetes dashboard and will land on the overview page for the default namespace. 


