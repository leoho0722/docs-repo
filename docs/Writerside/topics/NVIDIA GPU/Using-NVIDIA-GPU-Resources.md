# Using NVIDIA GPU Resources

**Table of Contents**

- [Remove default driver](#remove-default-driver)
  - [Check the default driver is existing or not](#check-the-default-driver-is-existing-or-not)
  - [List the default driver](#list-the-default-driver)
  - [Delete the default driver and reboot](#delete-the-default-driver-and-reboot)
- [Install NVIDIA CUDA](#install-nvidia-cuda)
  - [Check NVIDIA CUDA](#check-nvidia-cuda)
- [Install NVIDIA cuDNN](#install-nvidia-cudnn)
- [Install DKMS](#install-dkms)
- [Using NVIDIA GPU resources on Docker](#using-nvidia-gpu-resources-on-docker)
  - [Prerequisites: Install NVIDIA Container Toolkit](#prerequisites-install-nvidia-container-toolkit)
  - [Configure Docker](#configure-docker)
- [Using NVIDIA GPU resources on Kubernetes](#using-nvidia-gpu-resources-on-kubernetes)
  - [Prerequisites: Install NVIDIA Container Toolkit](#prerequisites-install-nvidia-container-toolkit_1)
  - [Configure containerd (for Kubernetes)](#configure-containerd-for-kubernetes)
  - [Install Method](#install-method)
    - [Method 1: Install NVIDIA GPU Operator](#method-1-install-nvidia-gpu-operator)
    - [Method 2: Install Kubernetes NVIDIA Device Plugin](#method-2-install-kubernetes-nvidia-device-plugin)
  - [Validation](#validation)
    - [Check Pod can run GPU Jobs or not](#check-pod-can-run-gpu-jobs-or-not)
    - [Check node can use GPU resource or not](#check-node-can-use-gpu-resource-or-not)

## Remove default driver

### Check the default driver is existing or not

```Shell
sudo lshw -C display
```

![截圖 2024-04-12 15.59.08.png](截圖_2024-04-12_15.59.08.png)

### List the default driver

```Shell
lsmod | grep nouveau
```

If "nouveau" appears, it means there is a default driver.

![截圖 2024-04-12 15.59.51.png](截圖_2024-04-12_15.59.51.png)

### Delete the default driver and reboot

```Shell
cat <<EOF | sudo tee /etc/modprobe.d/blacklist-nouveau.conf
blacklist nouveau
options nouveau modeset=0
EOF
sudo update-initramfs -u
sudo reboot
```

![截圖 2024-04-12 16.02.24.png](截圖_2024-04-12_16.02.24.png)

## Install NVIDIA CUDA

```Shell
sudo apt-get update -y
sudo apt install -y build-essential linux-headers-$(uname -r) wget
```

Download the required CUDA Toolkit version from the NVIDIA official website

[NVIDIA CUDA Toolkit Official Download Website](https://developer.nvidia.com/cuda-toolkit-archive)

![截圖 2024-04-12 16.12.08.png](截圖_2024-04-12_16.12.08.png)

The environment demonstrated here is

* CUDA Toolkit 12.4.1
* Ubuntu 22.04 x86_64
* runfile (local) Installer Type

![截圖 2024-04-12 16.13.46.png](截圖_2024-04-12_16.13.46.png)

```Shell
wget https://developer.download.nvidia.com/compute/cuda/12.4.1/local_installers/cuda_12.4.1_550.54.15_linux.run
sudo sh cuda_12.4.1_550.54.15_linux.run
```

After execution, you will see a UI-like installation menu.
Enter accept to accept the terms of use

![截圖 2024-04-12 16.49.14.png](截圖_2024-04-12_16.49.14.png)

Use the space bar to select "Driver" and "CUDA Toolkit"

![截圖 2024-04-12 16.54.35.png](截圖_2024-04-12_16.54.35.png)

After the installation is complete, add the following two lines to the end of ```~/.bashrc```

```Shell
nano ~/.bashrc
```

```Shell
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
```

```Shell
source ~/.bashrc
```

![截圖 2024-04-12 17.01.39.png](截圖_2024-04-12_17.01.39.png)

![截圖 2024-04-12 17.00.37.png](截圖_2024-04-12_17.00.37.png)

### Check NVIDIA CUDA

```Shell
nvidia-smi
nvcc --version
```

![截圖 2024-04-12 17.13.18.png](截圖_2024-04-12_17.13.18.png)

![截圖 2024-04-12 17.32.46.png](截圖_2024-04-12_17.32.46.png)

## Install NVIDIA cuDNN

Download the required cuDNN version from the NVIDIA official website

[NVIDIA cuDNN Official Download Website](https://developer.download.nvidia.com/compute/cudnn/redist/cudnn/)

![截圖 2024-04-12 17.19.22.png](截圖_2024-04-12_17.19.22.png)

The system environment here is Ubuntu 22.04 x86_64, so choose ```linux-x86_64/```

![截圖 2024-04-12 17.21.06.png](截圖_2024-04-12_17.21.06.png)

Here we take ```cudnn-linux-x86_64-8.9.7.29_cuda12-archive.tar.xz``` as an example

```Shell
wget https://developer.download.nvidia.com/compute/cudnn/redist/cudnn/linux-x86_64/cudnn-linux-x86_64-8.9.7.29_cuda12-archive.tar.xz
tar -xvf cudnn-linux-x86_64-8.9.7.29_cuda12-archive.tar.xz
sudo cp cudnn-linux-x86_64-8.9.7.29_cuda12-archive/include/cudnn*.h /usr/local/cuda/include/
sudo cp -P cudnn-linux-x86_64-8.9.7.29_cuda12-archive/lib/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/include/ /usr/local/cuda/lib64
cat /usr/local/cuda/include/cudnn_version.h | grep CUDNN_MAJOR -A 2
```

![截圖 2024-04-12 17.24.56.png](截圖_2024-04-12_17.24.56.png)

![截圖 2024-04-12 17.25.41.png](截圖_2024-04-12_17.25.41.png)

![截圖 2024-04-12 17.30.50.png](截圖_2024-04-12_17.30.50.png)

## Install DKMS

```Shell
sudo apt-get update -y
sudo apt install -y dkms

# NVIDIA Driver Version 可以透過 nvidia-smi 取得，例如：550.54.15
sudo dkms install -m nvidia -v <NVIDIA Driver Version>
```

## Using NVIDIA GPU resources on Docker

### Prerequisites: Install NVIDIA Container Toolkit

[NVIDIA Container Toolkit Official Installation Guide](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)

#### Installing with Apt

1. Configure the production repository

```Shell
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

2. Update the packages list from the repository

```Shell
sudo apt-get update
```

3. Install the NVIDIA Container Toolkit packages

```Shell
sudo apt-get install -y nvidia-container-toolkit
```

![截圖 2024-04-12 17.47.34.png](截圖_2024-04-12_17.47.34.png)

### Configure Docker

```Shell
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

Confirm whether ```/etc/docker/daemon.json``` has correctly configured NVIDIA GPU runtime, similar to the following information

```Shell
cat /etc/docker/daemon.json
```

```json
{
    "exec-opts": [
      "native.cgroupdriver=systemd"
    ],
    "log-driver": "json-file",
    "log-opts": {
      "max-size": "100m"
    },
    "default-runtime": "nvidia",
    "runtimes": {
      "nvidia": {
        "args": [],
        "path": "nvidia-container-runtime"
      }
    },
    "storage-driver": "overlay2"
}
```

If you follow the official steps but do not automatically set ```default-runtime``` to ```nvidia```, you need to manually add it.

```Shell
sudo nano /etc/docker/daemon.json
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## Using NVIDIA GPU resources on Kubernetes

### Prerequisites: Install NVIDIA Container Toolkit {id="prerequisites-install-nvidia-container-toolkit_1"}

[NVIDIA Container Toolkit Official Installation Guide](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)

#### Installing with Apt {id="installing-with-apt_1"}

1. Configure the production repository

```Shell
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

2. Update the packages list from the repository

```Shell
sudo apt-get update
```

3. Install the NVIDIA Container Toolkit packages

```Shell
sudo apt-get install -y nvidia-container-toolkit
```

![截圖 2024-04-12 17.47.34.png](截圖_2024-04-12_17.47.34.png)

### Configure containerd (for Kubernetes)

Before execute NVIDIA containerd for Kubernetes configure command, copy original containerd config.toml (in `/etc/containerd`) file to current directory first.

```Shell
sudo cp /etc/containerd/config.toml ./config.toml
```

Then, execute NVIDIA containerd for Kubernetes configure command

```Shell
sudo nvidia-ctk runtime configure --runtime=containerd
```

Next, copy content of `/etc/containerd/config.toml` into config.toml in current directory

Such as below of `[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia]` and `[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia.options]` section

```Text
[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
  ...
  
    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "runc"
      ...
      
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
      ...
      
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia]
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"
  
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia.options]
            BinaryName = "/usr/bin/nvidia-container-runtime"
            SystemdCgroup = true
```

Next, update `default_runtime_name` from `runc` to `nvidia`.

Such as below `[plugins."io.containerd.grpc.v1.cri".containerd]` section

```Text
[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
  ...
  
    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "nvidia"
      ...
      
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
      ...
      
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia]
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"
  
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia.options]
            BinaryName = "/usr/bin/nvidia-container-runtime"
            SystemdCgroup = true
```

Finally, restart containerd service

```Shell
sudo systemctl restart containerd
```

### Install Method

#### Method 1: Install NVIDIA GPU Operator {collapsible="true"}

[NVIDIA GPU Operator Official Documentation](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/getting-started.html)

##### Prerequisites: Install Helm

```Shell
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 \
    && chmod 700 get_helm.sh \
    && ./get_helm.sh
```

##### Add the NVIDIA Helm repository

```Shell
helm repo add nvidia https://helm.ngc.nvidia.com/nvidia \
    && helm repo update
```

##### Install the GPU Operator

###### Option 1: Install the Operator with the default configuration {collapsible="true"}

```Shell
helm install --wait --generate-name \
    -n gpu-operator --create-namespace \
    nvidia/gpu-operator \
    --version=v24.9.0
```

###### Option 2: Install the Operator with the specify version {collapsible="true"}

GPU Operator Version and dependent version list of related Components

| GPU Operator Version | CUDA Version | Driver Version | Container Toolkit Version | Device Plugin Version |
|----------------------|--------------|----------------|---------------------------|-----------------------|
| v24.9.0              | 12.6.2       | 550.127.05     | 1.17.0                    | 0.17.0                |
| v24.6.2              | 12.6.1       | 550.90.07      | 1.16.2                    | 0.16.2                |
| v24.6.1              | 12.5.1       | 550.90.07      | 1.16.1                    | 0.16.2                |
| v24.6.0              | 12.5.1       | 550.90.07      | 1.16.1                    | 0.16.1                |
| v24.3.0              | 12.4.1       | 550.54.15      | 1.15.0                    | 0.15.0                |
| v23.9.2              | 12.3.2       | 550.54.14      | 1.14.6                    | 0.14.5                |
| v23.9.1              | 12.3.1       | 535.129.03     | 1.14.3                    | 0.14.3                |
| v23.9.0              | 12.2.2       | 535.104.12     | 1.14.3                    | 0.14.2                |
| v23.6.2              | 12.3.1       | 535.104.05     | 1.13.4                    | 0.14.1                |
| v23.6.1              | 12.2.0       | 535.104.05     | 1.13.4                    | 0.14.1                |
| v23.6.0              | 12.2.0       | 535.86.10      | 1.13.4                    | 0.14.1                |
| v23.3.2              | 12.1.1       | 525.105.17     | 1.13.0                    | 0.14.0                |

```Shell
export GPU_OPERATOR_VERSION=v24.9.0
```

```Shell
helm install --wait --generate-name \
    -n gpu-operator --create-namespace \
    nvidia/gpu-operator \
    --version=$GPU_OPERATOR_VERSION
```

###### Option 3: Pre-Installed NVIDIA GPU Drivers {collapsible="true"}

```Shell
helm install --wait --generate-name \
     -n gpu-operator --create-namespace \
     nvidia/gpu-operator \
     --version=v24.9.0 \
     --set driver.enabled=false
```

###### Option 4: Pre-Installed NVIDIA GPU Drivers and NVIDIA Container Toolkit {collapsible="true"}

```Shell
helm install --wait --generate-name \
     -n gpu-operator --create-namespace \
     nvidia/gpu-operator \
     --version=v24.9.0 \
     --set driver.enabled=false \
     --set toolkit.enabled=false
```

#### Method 2: Install Kubernetes NVIDIA Device Plugin {collapsible="true"}

[Kubernetes NVIDIA Device Plugin Official GitHub Repo](https://github.com/NVIDIA/k8s-device-plugin)

Deploy ```nvidia-device-plugin``` DaemonSet to Kubernetes Cluster

```Shell
kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.17.0/deployments/static/nvidia-device-plugin.yml
```

Check the status of NVIDIA Device Plugin DaemonSet and Pods in a Kubernetes cluster

```Shell
kubectl get ds -n kube-system
kubectl get pods -n kube-system
```

![Check the status of NVIDIA Device Plugin DaemonSet and Pods in a Kubernetes cluster](check_nvidia_device_plugin_ds_pod_in_kubesystem.png)

### Validation

#### Check Pod can run GPU Jobs or not

##### Using NVIDIA GPU Operator {collapsible="true"}

Get the Pods of gpu-operator namespace in all Worker nodes

```Shell
kubectl get pod -n gpu-operator -o wide
```

![Get the Pods of gpu-operator namespace in all Worker nodes](check_pod_on_gpu_operator_namespace_using_gpu_operator.png)

View the log output of Pod nvidia-cuda-validator deployed in all Worker nodes

![View the log output of Pod nvidia-cuda-validator deployed in all Worker nodes](view_pod_log_output_check_can_use_gpu_resources_using_gpu_operator.png)

Outputting ```cuda workload validation is successful``` means that GPU resources are successfully used in the Pod.

##### Using Kubernetes NVIDIA Device Plugin {collapsible="true"}

```Shell
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  restartPolicy: Never
  containers:
    - name: cuda-container
      image: nvcr.io/nvidia/k8s/cuda-sample:vectoradd-cuda10.2
      resources:
        limits:
          nvidia.com/gpu: 1 # requesting 1 GPU
  tolerations:
  - key: nvidia.com/gpu
    operator: Exists
    effect: NoSchedule
EOF
```

```Shell
kubectl logs pod/gpu-pod
```

Outputting ```Test PASSED``` means that GPU resources are successfully used in the Pod.

![截圖 2024-04-12 18.30.15.png](截圖_2024-04-12_18.30.15.png)

#### Check node can use GPU resource or not

##### Using NVIDIA GPU Operator {collapsible="true" id="using-nvidia-gpu-operator_1"}

```Shell
kubectl get nodes -o wide

kubectl describe node <node name> | grep nvidia.com

# Example
kubectl describe node ubuntu-d830mt | grep nvidia.com
kubectl describe node ubuntu-ms-7d98 | grep nvidia.com
```

Check if the node is labeled with the following labels

![Check if the node is labeled with the following labels](check_node_is_labeled_following_labels_using_gpu_operator.png)

##### Using Kubernetes NVIDIA Device Plugin {collapsible="true" id="using-kubernetes-nvidia-device-plugin_1"}

Check whether ```Capacity``` and ```Allocatable``` are displayed ```nvidia.com/gpu```

```Shell
kubectl describe node <node name>

# Example
kubectl describe node ubuntu3070ti
```

![截圖 2024-04-12 18.46.28.png](截圖_2024-04-12_18.46.28.png)