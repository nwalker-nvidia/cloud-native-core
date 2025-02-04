<h1>NVIDIA Cloud Native Core v6.2 - Install Guide for Developers</h1>
<h2>Introduction</h2>

This document describes how to setup the NVIDIA Cloud Native Core collection on a single or multiple systems. NVIDIA Cloud Native Core can be configured to create a single node Kubernetes cluster or to create/add additional worker nodes to join an existing cluster.

NVIDIA Cloud Native Core v6.2 includes:
- Ubuntu 20.04.4 LTS
- Containerd 1.6.5
- Kubernetes version 1.23.8
- Helm 3.8.2
- NVIDIA GPU Driver: 515.48.07
- NVIDIA Container Toolkit: 1.10.0
- NVIDIA GPU Operator 1.11.0
  - NVIDIA K8S Device Plugin: 0.12.2
  - NVIDIA DCGM-Exporter: 2.4.5-2.6.7
  - NVIDIA DCGM: 2.4.5-1
  - NVIDIA GPU Feature Discovery: 0.6.1
  - NVIDIA K8s MIG Manager: 0.4.2
  - NVIDIA Driver Manager: 0.4.0
  - Node Feature Discovery: 0.10.1

<h2>Table of Contents</h2>

- [Prerequisites](#Prerequisites)
- [Installing the Ubuntu Operating System](#Installing-the-Ubuntu-Operating-System)
- [Installing NVIDIA Driver](#(Installing-NVIDIA-Driver)
- [Installing Docker and Nvidia Container Toolkit](#Installing-Docker-and-Nvidia-Container-Toolkit)
- [Update the Docker Default Runtime](#Update-the-Docker-Default-Runtime)
- [Installing Containerd](#Installing-Containerd)
- [Installing Kubernetes](#Installing-Kubernetes)
- [Installing Helm](#Installing-Helm)
- [Adding an Additional Node to NVIDIA Cloud Native Core](#Adding-additional-node-to-NVIDIA-Cloud-Native-Core)
- [Installing the GPU Operator](#Installing-the-GPU-Operator)
- [Validating the GPU Operator](#Validating-the-GPU-Operator)
- [Build Docker Images and Deploy on Cloud Native Core](#Build-Docker-Images-and-Deploy-on-Cloud-Native-Core) 
- [Validate NVIDIA Cloud Native Core with an Application from NGC](#Validate-NVIDIA-Cloud-Native-Core-with-an-application-from-NGC)
- [Uninstalling the GPU Operator](#Uninstalling-the-GPU-Operator)

### Prerequisites
 
The following instructions assume the following:

- You have [NVIDIA-Certified Systems](https://docs.nvidia.com/ngc/ngc-deploy-on-premises/nvidia-certified-systems/index.html)
- You will perform a clean install.

To determine if your system qualifies as an NVIDIA Certified System, review the list of NVIDIA Certified Systems [here](https://docs.nvidia.com/ngc/ngc-deploy-on-premises/nvidia-certified-systems/index.html). 

Please note that NVIDIA Cloud Native Core is validated only on systems with the default kernel (not HWE).

### Installing the Ubuntu Operating System
These instructions require having Ubuntu Server LTS 20.04.4 on your system. The Ubuntu Server can be downloaded from http://cdimage.ubuntu.com/releases/20.04.4/release/.

For more information on installing Ubuntu server please reference the [Ubuntu Server Installation Guide](https://ubuntu.com/tutorials/tutorial-install-ubuntu-server#1-overview).

### Installing NVIDIA Driver 

Add NVIDIA Apt key

```
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/7fa2af80.pub
```
```
sudo apt-key add ./7fa2af80.pub
```

Install NVIDIA TRD Driver

```
sudo apt update 
```
```
wget https://us.download.nvidia.com/tesla/510.47.03/nvidia-driver-local-repo-ubuntu2004-510.47.03_1.0-1_amd64.deb
```
```
sudo apt install ./nvidia-driver-local-repo-ubuntu2004-510.47.03_1.0-1_amd64.deb -y
```

Update package index:

```
sudo apt update
```

Install Cuda Drivers

```
sudo apt install cuda-drivers -y
```

Once the NVIDIA Drivers installed, please reboot the system and run the below command to validate NVIDIA drivers are loaded 

```
nvidia-smi
```

Expected Output:

```
Wed May 11 12:47:29 2022
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 510.60.02    Driver Version: 510.60.02    CUDA Version: 11.6     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Quadro RTX 6000     Off  | 00000000:65:00.0  On |                  Off |
| 33%   32C    P8    30W / 260W |    154MiB / 24576MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A      1236      G   /usr/lib/xorg/Xorg                 39MiB |
|    0   N/A  N/A      2278      G   /usr/lib/xorg/Xorg                 58MiB |
|    0   N/A  N/A      2405      G   /usr/bin/gnome-shell               46MiB |
+-----------------------------------------------------------------------------+
```


### Installing Docker and NVIDIA Container Runtime

#### Installing Docker-CE

Set up the repository and update the apt package index:

```
$ sudo apt-get update
```

Install packages to allow apt to use a repository over HTTPS:

```
$ sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

Add Docker's official GPG key:

```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Verify that you now have the key with the fingerprint 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88 by searching for the last 8 characters of the fingerprint:
```
$ sudo apt-key fingerprint 0EBFCD88
    
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
``` 

Use the following command to set up the stable repository:

```
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

Install Docker Engine - Community
Update the apt package index:

```
$ sudo apt-get update
```

Install Docker Engine:

```
$ sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

Verify that Docker Engine - Community is installed correctly by running the hello-world image:

```
$ sudo docker run hello-world
```

More information on how to install Docker can be found at https://docs.docker.com/install/linux/docker-ce/ubuntu/. 

#### Installing NVIDIA Container Toolkit

Setup the pacakge repository 

```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
      && curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
      && curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
            sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
            sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

Update the package index

```
sudo apt update
```

Install NVIDIA Conatiner Toolkit

```
sudo apt-get install -y nvidia-docker2
```


### Update the Docker Default Runtime


Edit the docker daemon configuration to add the following line and save the file:

```
"default-runtime" : "nvidia"
```

Example: 
```
$ sudo nano /etc/docker/daemon.json
 
{
   "runtimes": {
   	"nvidia": {
       	"path": "nvidia-container-runtime",
           "runtimeArgs": []
   	}
   },
   "default-runtime" : "nvidia"
}
```

Now execute the below commands to restart the docker daemon:
```
sudo systemctl daemon-reload && sudo systemctl restart docker
```

#### Validate docker default runtime

Execute the below command to validate docker default runtime as NVIDIA:

```
$ sudo docker info | grep -i runtime
```

Output:
```
Runtimes: nvidia runc
Default Runtime: nvidia
```


### Installing Containerd

Set up the repository and update the apt package index:

```
 sudo apt-get update
```

Install packages to allow apt to use a repository over HTTPS:

```
 sudo apt-get install -y apt-transport-https gnupg-agent libseccomp2 autotools-dev debhelper software-properties-common
```

Configure the prerequisites for Containerd:

```
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```

```
 sudo modprobe overlay
```
```
 sudo modprobe br_netfilter
```

Setup required sysctl params; these persist across reboots:
```
 cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```

Apply sysctl params without reboot:
```
 sudo sysctl --system
```

Download the Containerd tarball:

```
 wget https://github.com/containerd/containerd/releases/download/v1.6.5/cri-containerd-cni-1.6.5-linux-amd64.tar.gz
```
```
 sudo tar --no-overwrite-dir -C / -xzf cri-containerd-cni-1.6.5-linux-amd64.tar.gz
```
```
 rm -rf cri-containerd-cni-1.6.5-linux-amd64.tar.gz
```

Install Containerd:
```
 sudo mkdir -p /etc/containerd
```
```
 containerd config default | sudo tee /etc/containerd/config.toml
```
```
 sudo systemctl enable containerd && sudo systemctl restart containerd
```

For additional information on installing Containerd, please reference [Install Containerd with Release Tarball](https://github.com/containerd/containerd/blob/master/docs/cri/installation.md). 

### Installing Kubernetes 

Make sure Containerd has been started and enabled before beginning installation:

```
 sudo systemctl start containerd && sudo systemctl enable containerd
```

Execute the following to add apt keys:

```
 sudo apt-get update && sudo apt-get install -y apt-transport-https curl
```
```
 curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```
```
 sudo mkdir -p  /etc/apt/sources.list.d/
```

Create kubernetes.list:

```
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
```

Now execute the below to install kubelet, kubeadm, and kubectl:

```
 sudo apt-get update
```
```
 sudo apt-get install -y -q kubelet=1.23.8-00 kubectl=1.23.8-00 kubeadm=1.23.8-00
```
```
 sudo apt-mark hold kubelet kubeadm kubectl
```

Create a kubelet default with Containerd:
```
cat <<EOF | sudo tee /etc/default/kubelet
KUBELET_EXTRA_ARGS=--cgroup-driver=systemd --container-runtime=remote --runtime-request-timeout=15m --container-runtime-endpoint="unix:/run/containerd/containerd.sock"
EOF
```

Reload the system daemon:
```
 sudo systemctl daemon-reload
```

Disable swap:
```
 sudo swapoff -a
```
```
 sudo nano /etc/fstab
```

`NOTE:` Add a # before all the lines that start with /swap. # is a comment, and the result should look something like this:

```
UUID=e879fda9-4306-4b5b-8512-bba726093f1d / ext4 defaults 0 0
UUID=DCD4-535C /boot/efi vfat defaults 0 0
#/swap.img       none    swap    sw      0       0
```

#### Initializing the Kubernetes cluster to run as a control-plane node

Execute the following command:

```
 sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --cri-socket=/run/containerd/containerd.sock --kubernetes-version="v1.23.5"
```

Output:
```
Your Kubernetes control-plane has initialized successfully!
 
To start using your cluster, you need to run the following as a regular user:
 
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
 
Alternatively, if you are the root user, you can run:
 
  export KUBECONFIG=/etc/kubernetes/admin.conf
 
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/
 
Then you can join any number of worker nodes by running the following on each as root:
 
kubeadm join <your-host-IP>:6443 --token 489oi5.sm34l9uh7dk4z6cm \
        --discovery-token-ca-cert-hash sha256:17165b6c4a4b95d73a3a2a83749a957a10161ae34d2dfd02cd730597579b4b34
```


Following the instructions in the output, execute the commands as shown below:

```
 mkdir -p $HOME/.kube
 sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

With the following command, you install a pod-network add-on to the control plane node. We are using calico as the pod-network add-on here:

```
 kubectl apply -f https://docs.projectcalico.org/v3.21/manifests/calico.yaml 
```

Update the Calico Daemonset 

```
kubectl set env daemonset/calico-node -n kube-system IP_AUTODETECTION_METHOD=interface=ens\*,eth\*,enc\*,enp\*
```

You can execute the below commands to ensure that all pods are up and running:

```
 kubectl get pods --all-namespaces
```

Output:

```
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-65b8787765-bjc8h   1/1     Running   0          2m8s
kube-system   calico-node-c2tmk                          1/1     Running   0          2m8s
kube-system   coredns-5c98db65d4-d4kgh                   1/1     Running   0          9m8s
kube-system   coredns-5c98db65d4-h6x8m                   1/1     Running   0          9m8s
kube-system   etcd-#yourhost                             1/1     Running   0          8m25s
kube-system   kube-apiserver-#yourhost                   1/1     Running   0          8m7s
kube-system   kube-controller-manager-#yourhost          1/1     Running   0          8m3s
kube-system   kube-proxy-6sh42                           1/1     Running   0          9m7s
kube-system   kube-scheduler-#yourhost                   1/1     Running   0          8m26s
```

The get nodes command shows that the control-plane node is up and ready:

```
 kubectl get nodes
```

Output:

```
NAME             STATUS   ROLES                  AGE   VERSION
#yourhost        Ready    control-plane,master   10m   v1.23.8
```

Since we are using a single-node Kubernetes cluster, the cluster will not schedule pods on the control plane node by default. To schedule pods on the control plane node, we have to remove the taint by executing the following command:

```
 kubectl taint nodes --all node-role.kubernetes.io/master-
```

Refer to [Installing Kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
for more information.

### Installing Helm 

Execute the following command to download and install Helm 3.8.2: 

```
 wget https://get.helm.sh/helm-v3.8.2-linux-amd64.tar.gz && \
 tar -zxvf helm-v3.8.2-linux-amd64.tar.gz && \
 sudo mv linux-amd64/helm /usr/local/bin/helm && \ 
 rm -rf helm-v3.8.2-linux-amd64.tar.gz linux-amd64/
```

Refer to the Helm 3.8.1 [release notes](https://github.com/helm/helm/releases) and the [Installing Helm guide](https://helm.sh/docs/using_helm/#installing-helm) for more information.


### Adding an Additional Node to NVIDIA Cloud Native Core

`NOTE:` If you're not adding additional nodes, please skip this step and proceed to the next step [Installing NVIDIA Network Operator](#Installing-NVIDIA-Network-Operator)

Make sure to install the Containerd and Kubernetes packages on additional nodes.

Prerequisites: 
- [Installing Docker and NVIDIA Container Toolkit](#Installing-Docker-and-NVIDIA-Container-Toolkit)
- [Installing Containerd](#Installing-Containerd)
- [Installing Kubernetes](#Installing-Kubernetes)
- [Disable swap](#Disable-swap)

Once the prerequisites are completed on the additional nodes, execute the below command on the control-plane node and then execute the join command output on an additional node to add the additional node to NVIDIA Cloud Native Core:

```
 sudo kubeadm token create --print-join-command
```

Output:
```
example: 
sudo kubeadm join 10.110.0.34:6443 --token kg2h7r.e45g9uyrbm1c0w3k     --discovery-token-ca-cert-hash sha256:77fd6571644373ea69074dd4af7b077bbf5bd15a3ed720daee98f4b04a8f524e
```
`NOTE`: control-plane node and worker node should not have the same node name. 

The get nodes command shows that the master and worker nodes are up and ready:

```
 kubectl get nodes
```

Output:

```
NAME             STATUS   ROLES                  AGE   VERSION
#yourhost        Ready    control-plane,master   10m   v1.23.5
#yourhost-worker Ready                           10m   v1.23.5
```

### Installing GPU Operator

Add the NVIDIA repo:

```
 helm repo add nvidia https://helm.ngc.nvidia.com/nvidia
```

Update the Helm repo:

```
 helm repo update
```

Install GPU Operator:

`NOTE:` As we are preinstalled with NVIDIA Driver and NVIDIA Container Toolkit, we need to set as `false` when installing the GPU Operator

```
 helm install --version 1.10.1 --create-namespace --namespace nvidia-gpu-operator --devel nvidia/gpu-operator --set driver.enabled=false,toolkit.enabled=false --wait --generate-name
```

#### Validating the State of the GPU Operator:

Please note that the installation of the GPU Operator can take a couple of minutes. How long the installation will take depends on your internet speed.

```
kubectl get pods --all-namespaces | grep -v kube-system
```

```
NAMESPACE                NAME                                                              READY   STATUS      RESTARTS   AGE
default                  gpu-operator-1622656274-node-feature-discovery-master-5cddq96gq   1/1     Running     0          2m39s
default                  gpu-operator-1622656274-node-feature-discovery-worker-wr88v       1/1     Running     0          2m39s
default                  gpu-operator-7db468cfdf-mdrdp                                     1/1     Running     0          2m39s
gpu-operator-resources   gpu-feature-discovery-g425f                                       1/1     Running     0          2m20s
gpu-operator-resources   nvidia-cuda-validator-s6x2p                                       0/1     Completed   0          48s
gpu-operator-resources   nvidia-dcgm-exporter-wtxnx                                        1/1     Running     0          2m20s
gpu-operator-resources   nvidia-dcgm-jbz94                                                 1/1     Running     0          2m20s
gpu-operator-resources   nvidia-device-plugin-daemonset-hzzdt                              1/1     Running     0          2m20s
gpu-operator-resources   nvidia-device-plugin-validator-9nkxq                              0/1     Completed   0          17s
gpu-operator-resources   nvidia-operator-validator-cw4j5                                   1/1     Running     0          2m20s

```

Please refer to the [GPU Operator page](https://ngc.nvidia.com/catalog/helm-charts/nvidia:gpu-operator) on NGC for more information.

For multiple worker nodes, execute the below command to fix the CoreDNS and Node Feature Discovery. 

```
kubectl delete pods $(kubectl get pods -n kube-system | grep core | awk '{print $1}') -n kube-system; kubectl delete pod $(kubectl get pods -o wide -n gpu-operator-resources | grep node-feature-discovery | grep -v master | awk '{print $1}') -n gpu-operator-resources
```

#### GPU Operator with MIG

`NOTE:` Only A100 and A30 GPUs are supported for GPU Operator with MIG

Multi-Instance GPU (MIG) allows GPUs based on the NVIDIA Ampere architecture (such as NVIDIA A100) to be securely partitioned into separate GPU instances for CUDA applications. For more information about enabling the MIG capability, please refer to [GPU Operator with MIG](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/gpu-operator-mig.html) 


### Validating the GPU Operator

GPU Operator validates the  through the nvidia-device-plugin-validation pod and the nvidia-driver-validation pod. If both are completed successfully (see output from kubectl get pods --all-namespaces | grep -v kube-system), NVIDIA Cloud Native Core is working as expected. This section provides two examples of validating that the GPU is usable from within a pod to validate the  manually.

#### Example 1: nvidia-smi

Execute the following:

```
$ kubectl run nvidia-smi --rm -t -i --restart=Never --image=nvidia/cuda:11.6.0-base --limits=nvidia.com/gpu=1 -- nvidia-smi
```

Output:

``` 
Wed May 11 12:47:29 2022
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 510.60.02    Driver Version: 510.60.02    CUDA Version: 11.6     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Quadro RTX 6000     Off  | 00000000:65:00.0  On |                  Off |
| 33%   32C    P8    30W / 260W |    154MiB / 24576MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A      1236      G   /usr/lib/xorg/Xorg                 39MiB |
|    0   N/A  N/A      2278      G   /usr/lib/xorg/Xorg                 58MiB |
|    0   N/A  N/A      2405      G   /usr/bin/gnome-shell               46MiB |
+-----------------------------------------------------------------------------+
pod "nvidia-smi" deleted
```

#### Example 2: CUDA-Vector-Add

Create a pod YAML file:

```
$ cat <<EOF | tee cuda-samples.yaml
apiVersion: v1
kind: Pod
metadata:
  name: cuda-vector-add
spec:
  restartPolicy: OnFailure
  containers:
    - name: cuda-vector-add
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
EOF
```

Execute the below command to create a sample GPU pod:

```
$ kubectl apply -f cuda-samples.yaml
```

Confirm the cuda-samples pod was created:

```
$ kubectl get pods
``` 

NVIDIA Cloud Native Core works as expected if the get pods command shows the pod status as completed.

### Build Docker Images and Deploy on Cloud Native Core 
You can build the docker images using docker CLI but you can not use them directly on Cloud Native Core. Please find the below steps to import the docker image into Cloud Native Core. 

```
sudo docker save test-image:0.1.0 > test-image.tgz; sudo ctr -n=k8s.io images import test-image.tgz
```

### Validate NVIDIA Cloud Native Core with an Application from NGC
Another option to validate NVIDIA Cloud Native Core is by running a demo application hosted on NGC.

NGC is NVIDIA's GPU-optimized software hub. NGC provides a curated set of GPU-optimized software for AI, HPC, and visualization. The content provided by NVIDIA and third-party ISVs simplify building, customizing, and integrating GPU-optimized software into workflows, accelerating the time to solutions for users.

Containers, pre-trained models, Helm charts for Kubernetes deployments, and industry-specific AI toolkits with software development kits (SDKs) are hosted on NGC. For more information about how to deploy an application that is hosted on NGC or the NGC Private Registry, please refer to this [NGC Registry Guide](https://github.com/NVIDIA/cloud-native-core/blob/master/install-guides/NGC_Registry_Guide_v1.0.md). Visit the [public NGC documentation](https://docs.nvidia.com/ngc) for more information.

The steps in this section use the publicly available DeepStream - Intelligent Video Analytics (IVA) demo application Helm Chart. The application can validate the full NVIDIA Cloud Native Core and test the connectivity of NVIDIA Cloud Native Core to remote sensors. DeepStream delivers real-time AI-based video and image understanding and multi-sensor processing on GPUs. For more information, please refer to the [Helm Chart](https://ngc.nvidia.com/catalog/helm-charts/nvidia:video-analytics-demo).

There are two ways to configure the DeepStream - Intelligent Video Analytics Demo Application on your NVIDIA Cloud Native Core

- Using a camera
- Using the integrated video file (no camera required)

#### Using a camera

##### Prerequisites: 
- RTSP Camera stream

Go through the below steps to install the demo application:
```
1. helm fetch https://helm.ngc.nvidia.com/nvidia/charts/video-analytics-demo-0.1.8.tgz --untar

2. cd into the folder video-analytics-demo and update the file values.yaml

3. Go to the section Cameras in the values.yaml file and add the address of your IP camera. Read the comments section on how it can be added. Single or multiple cameras can be added as shown below

cameras:
 camera1: rtsp://XXXX
```

Execute the following command to deploy the demo application:
```
helm install video-analytics-demo --name-template iva
```

Once the Helm chart is deployed, access the application with the VLC player. See the instructions below. 

#### Using the integrated video file (no camera)

If you dont have a camera input, please execute the below commands to use the default video already integrated into the application:

```
$ helm fetch https://helm.ngc.nvidia.com/nvidia/charts/video-analytics-demo-0.1.8.tgz

$ helm install video-analytics-demo-0.1.8.tgz --name-template iva
```

Once the helm chart is deployed, access the application with the VLC player as per the below instructions. 
For more information about the demo application, please refer to the [application NGC page](https://ngc.nvidia.com/catalog/helm-charts/nvidia:video-analytics-demo)

#### Access from WebUI

Use the below WebUI URL to access the video analytic demo application from the browser:
```
http://IPAddress of Node:31115/
```

#### Access from VLC

Download VLC Player from https://www.videolan.org/vlc/ on the machine where you intend to view the video stream.

View the video stream in VLC by navigating to Media > Open Network Stream > Entering the following URL:

```
rtsp://IPAddress of Node:31113/ds-test
```

You should see the video output like below with the AI model detecting objects.

![Deepstream_Video](screenshots/Deepstream.png)

`NOTE:` Video stream in VLC will change if you provide an input RTSP camera.


### Uninstalling the GPU Operator 

Execute the below commands to uninstall the GPU Operator:

```
$ helm ls
NAME                    NAMESPACE                      REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
gpu-operator-1606173805 gpu-operator-resources         1               2022-05-11 20:23:28.063421701 +0000 UTC deployed        gpu-operator-1.10.1      1.10.1

$ helm del gpu-operator-1606173805 -n gpu-operator-resources
```
