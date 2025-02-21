---
title: "From Training to Serving: AI with PyTorch & Kubeflow on K8s (I)"
subtitle: "Set up a GPU Kubernetes cluster on GCP"
date: 2025-02-18T20:23:12-08:00
draft: true
---

I’m interested in covering the entire pipeline—from model training to model serving
using PyTorch, Kubeflow, and Kubernetes. In this blog series, I’ll walk through the
a basic example of that process, aimed at clarifying some of the moving parts involved.

For demonstration, I built a Kubernetes cluster on a GCP instance featuring a T4 GPU, 
2 vCPUs, and 8 GB of RAM. GCP is simply my choice here because I’ve got a $300 credit, 
but AWS or Azure would do the job just as well.

## Setup GPU Drivers and Tools

Follow the [instruction](https://cloud.google.com/compute/docs/gpus/install-drivers-gpu)
to install the GPU drivers on your node.

1. Download install script
```bash
curl -L https://github.com/GoogleCloudPlatform/compute-gpu-installation/releases/download/cuda-installer-v1.2.0/cuda_installer.pyz --output cuda_installer.pyz
```

2. Run the script
```bash
sudo python3 cuda_installer.pyz install_driver
```

3. Note that the node will be rebooted automatically during the installation. 
After it restarts, re-run the script to continue.

4. After installation completes, confirm that everything is correct:
```bash
sudo nvidia-smi
```
A successful installation will display your GPU details.

## Setup Docker and NVIDIA Container Toolkit
1. Remove any older Docker versions
```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

2. Install Docker prerequisites
```bash
sudo apt-get update
sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

3. Add Docker's GPG key
```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

4. Install Docker
```bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

5. Create a docker group
```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

6. Log out and log back in to apply the new group settings

7. Refer to the [instruction](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) 
to install the NVIDIA Container Toolkit

```bash
# Add the package repositories
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
 && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
   sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
   sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list   
```

```bash
sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
```

## Setup Kubernetes Cluster with GPU

1. Consult the [instruction](https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fdebian+package)
for installing Minikube. I chose Minikube for its straightforward GPU node setup.
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
sudo dpkg -i minikube_latest_amd64.deb
```

2. Refer to the [instruction](https://minikube.sigs.k8s.io/docs/tutorials/nvidia/)
to enable NVIDIA GPU support in Minikube.

1) Check if bpf_jit_harden is set to 0
```bash
sudo sysctl net.core.bpf_jit_harden
```
2) If it’s 1, update it to 0
```bash
echo "net.core.bpf_jit_harden=0" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

3) Reconfigure Docker
```bash
sudo nvidia-ctk runtime configure --runtime=docker && sudo systemctl restart docker
```

4) Launch Minikube with NVIDIA GPU support
```bash
minikube start --driver docker --container-runtime docker --gpus all
```

5) Install kubectl
```bash
sudo apt-get install -y kubectl
```

6) Create a GPU-enabled pod
```bash
kubectl apply -f- <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: gpu-test
spec:
  containers:
  - name: cuda-test
    image: nvidia/cuda:11.6.2-base-ubuntu20.04
    resources:
      limits:
        nvidia.com/gpu: 1
    command: ["nvidia-smi"]
  restartPolicy: OnFailure
EOF
```

7) Check its logs
```bash
kubectl logs gpu-test
```
You should see GPU details if the pod is running correctly.
