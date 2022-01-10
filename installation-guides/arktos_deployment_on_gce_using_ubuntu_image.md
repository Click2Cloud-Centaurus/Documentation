# Setup a multi-node Arktos cluster using Ubuntu or custom image on GCE

This document outlines the steps to deploy arktos cluster on GCE from remote workstation machine. User will need to run following steps on workstation machine (the recommended instance size should be atleast ```16 CPU and 32GB RAM``` and the storage size should be ```150GB``` or more)


This document outline steps to deploy arktos cluster on GCE from a remote workstation machine. You need to run the following steps on a workstation machine (recommended instance size should be at least **16 CPU and 32GB RAM** and **storage size should be 150GB or more**)
### Prerequisites

1. User will need a GCP account, and [gcloud](https://cloud.google.com/sdk/docs/install#deb) configured in your bash profile. Please refer to gcloud configuration documentation or you can use following steps to install and configure gcloud utility.
#### Install gcloud
```bash
echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
sudo apt-get install apt-transport-https ca-certificates gnupg -y -q
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -
sudo apt-get update -y && sudo apt-get install make google-cloud-sdk -y
gcloud init # Provide credentials
```
2. User will need docker and golang to create binaries and docker images.

#### Install Docker 
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update -y
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
```

#### Install Golang
```bash
wget https://storage.googleapis.com/golang/go1.15.4.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.15.4.linux-amd64.tar.gz
sudo echo 'export PATH=$PATH:/usr/local/go/bin' >> $HOME/.profile
sudo echo 'export GOPATH=$HOME/gopath' >> $HOME/.profile
source $HOME/.profile
```

### Deploy Arktos cluster

#### Clone arktos repository
```bash
mkdir -p $HOME/go/src/k8s.io
cd $HOME/go/src/k8s.io
git clone -b  ubuntu-image-fix https://github.com/Click2Cloud-Centaurus/arktos.git

```
#### Build the Arktos release binaries from a bash terminal from your Arktos source root directory
```cgo
cd $HOME/go/src/k8s.io/arktos
sudo make clean
sudo make quick-release
```

#### Modify `cluster/gce/config-default.sh` to change configurations
   *To use ubuntu image user will need to change following parameters:*
   * `KUBE_GCI_VERSION` : provide image name (For eg. `ubuntu-1804-bionic-v20210928`)
   * `KUBE_GCE_MASTER_PROJECT`: provide project name(for ubuntu image use `ubuntu-os-cloud`)
   * `KUBE_GCE_NODE_PROJECT`: s project name(for ubuntu image use `ubuntu-os-cloud`)


#### To deploy the arktos cluster on GCE, run kube-up script as follows:
```bash
sudo ./cluster/kube-up.sh
```
kube-up script displays the admin cluster details upon successful deployment.

## Using the arktos cluster using kubectl

To use arktos cluster, use kubectl utility. The build node is setup with the config to access the arktos cluster. e.g:
```bash
sudo ./cluster/kubectl.sh get nodes -o wide
```

## Arktos cluster tear-down

To terminate arktos cluster, run the following:
```bash
sudo ./cluster/kube-down.sh
```
