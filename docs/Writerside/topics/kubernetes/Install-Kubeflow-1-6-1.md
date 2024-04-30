# Install Kubeflow 1.6.1

**Table of Contents**

- [Prerequisites](#prerequisites)
- [Download and unzip Kubeflow 1.6.1 manifests](#download-and-unzip-kubeflow-1-6-1-manifests)
  - [Install zip via apt](#install-zip-via-apt)
  - [Download Kubeflow 1.6.1 manifests](#download-kubeflow-1-6-1-manifests)
  - [Unzip Kubeflow 1.6.1 manifests](#unzip-kubeflow-1-6-1-manifests)
- [Download Kustomize 3.2.0](#download-kustomize-3-2-0)
- [Install Kubeflow to Kubernetes](#install-kubeflow-to-kubernetes)
  - [Prepare](#prepare)
    - [PV for Kubernetes](#pv-for-kubernetes)
    - [Patch Kubeflow manifests](#patch-kubeflow-manifests)
  - [Istio](#istio)
  - [Knative Serving](#knative-serving)
  - [Cert Manager](#cert-manager)
  - [KServe](#kserve)
  - [KServe Built-in ClusterServingRuntimes](#kserve-built-in-clusterservingruntimes)
  - [Other](#other)
    - [Dex](#dex)
    - [OIDC AuthService](#oidc-authservice)
    - [Kubeflow Namespace](#kubeflow-namespace)
    - [Kubeflow Roles](#kubeflow-roles)
    - [Kubeflow Istio Resources](#kubeflow-istio-resources)
    - [Kubeflow Pipelines](#kubeflow-pipelines)
    - [katib](#katib)
    - [Central Dashboard](#central-dashboard)
    - [Admission Webhook](#admission-webhook)
    - [Notebooks](#notebooks)
    - [Profiles + KFAM](#profiles-kfam)
    - [Volumes Web App](#volumes-web-app)
    - [Tensorboard](#tensorboard)
    - [Training Operator](#training-operator)
    - [User Namespace](#user-namespace)

## Prerequisites

[Kubeflow 1.6.1 Official GitHub Repository](https://github.com/kubeflow/manifests/tree/v1.6.0)

* Kubernetes (up to 1.22) with default StorageClass
  * [Kubernetes 1.23 install script](https://raw.githubusercontent.com/leoho0722/airflow-mnist-example/kubeflow/scripts/install-kubernetes-v1.23-goldshadow.sh)
* Kustomize (version 3.2.0)
* kubectl

## Download and unzip Kubeflow 1.6.1 manifests

### Install zip via apt

```Shell
cd ~
sudo apt-get update
sudo apt-get install -y zip
```

### Download Kubeflow 1.6.1 manifests

```Shell
wget -O manifests-1.6.1.zip https://github.com/kubeflow/manifests/archive/refs/tags/v1.6.1.zip
```

### Unzip Kubeflow 1.6.1 manifests

```Shell
unzip manifests-1.6.1.zip -d .
```

## Download Kustomize 3.2.0

```Shell
cd ./manifests-1.6.1
wget -O kustomize https://github.com/kubernetes-sigs/kustomize/releases/download/v3.2.0/kustomize_3.2.0_linux_amd64
```

## Install Kubeflow to Kubernetes

### Prepare

```Shell
sudo rm -rf /mnt
sudo mkdir -p /mnt/minio # Others will be created automatically.
```

#### PV for Kubernetes

Save the following yaml content as ```kubeflow-pv.yaml``` and store it in the kubeflow manifests directory

```yaml
# Notice: Save this file in the kubeflow manifests directory

kind: PersistentVolume
apiVersion: v1
metadata:
  name: katib-mysql-pv
  namespace: kubeflow
  labels:
    type: local
    name: katib-mysql
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/katib-mysql"
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: minio-pv
  namespace: kubeflow
  labels:
    type: local
    name: kubeflow
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/minio"
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: mysql-pv
  namespace: kubeflow
  labels:
    type: local
    name: kubeflow
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/mysql"
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: authservice-pv
  namespace: istio-system
  labels:
    type: local
    name: authservice
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/autoservice"
```

#### Default StorageClass

Save the following yaml content as ```default-storageclass.yaml``` and store it in the kubeflow manifests directory

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
  name: standard
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

```Shell
kubectl apply -f default-storageclass.yaml
```

#### Patch Kubeflow manifests

Replace the original yaml file with the following content ↓

Reference: https://blog.csdn.net/qq_39698985/article/details/123981927

Original file path: ```manifests-1.6.1/common/oidc-authservice/base/statefulset.yaml```

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: authservice
spec:
  replicas: 1
  selector:
    matchLabels:
      app: authservice
  serviceName: authservice
  template:
    metadata:
      annotations:
        sidecar.istio.io/inject: "false"
      labels:
        app: authservice
    spec:
      # ===== FIXED PERSSIONS START =====
      # Fix Error opening bolt store: open /var/lib/authservice/data.db: permission denied
      initContainers:
        - name: fix-permissions
          image: busybox
          command: ["sh", "-c"]
          args: ["chmod -R 777 /var/lib/authservice;"]
          volumeMounts:
            - name: data
              mountPath: /var/lib/authservice
      # ===== FIXED PERSSIONS END =====
      containers:
        - name: authservice
          image: gcr.io/arrikto/kubeflow/oidc-authservice:6ac9400
          imagePullPolicy: Always
          ports:
            - name: http-api
              containerPort: 8080
          envFrom:
            - secretRef:
                name: oidc-authservice-client
            - configMapRef:
                name: oidc-authservice-parameters
          volumeMounts:
            - name: data
              mountPath: /var/lib/authservice
          readinessProbe:
            httpGet:
              path: /
              port: 8081
      securityContext:
        fsGroup: 111
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: authservice-pvc
```

### Istio

```Shell
./kustomize build common/istio-1-14/istio-crds/base | kubectl apply -f -
./kustomize build common/istio-1-14/istio-namespace/base | kubectl apply -f -
./kustomize build common/istio-1-14/istio-install/base | kubectl apply -f -
```

### Knative Serving

```Shell
./kustomize build common/knative/knative-serving/overlays/gateways | kubectl apply -f -
./kustomize build common/istio-1-14/cluster-local-gateway/base | kubectl apply -f -
```

### Cert Manager

```Shell
./kustomize build common/cert-manager/cert-manager/base | kubectl apply -f -
./kustomize build common/cert-manager/kubeflow-issuer/base | kubectl apply -f -
```

### KServe

```Shell
kubectl apply -f https://github.com/kserve/kserve/releases/download/v0.8.0/kserve.yaml
```

### KServe Built-in ClusterServingRuntimes

```Shell
kubectl apply -f https://github.com/kserve/kserve/releases/download/v0.8.0/kserve-runtimes.yaml
```

### Other

#### Dex

```Shell
./kustomize build common/dex/overlays/istio | kubectl apply -f -
```

#### OIDC AuthService

```Shell
./kustomize build common/oidc-authservice/base | kubectl apply -f -
kubectl apply -f kubeflow-pv.yaml -l name=authservice
```

#### Kubeflow Namespace

```Shell
./kustomize build common/kubeflow-namespace/base | kubectl apply -f -
```

#### Kubeflow Roles

```Shell
./kustomize build common/kubeflow-roles/base | kubectl apply -f -
```

#### Kubeflow Istio Resources

```Shell
./kustomize build common/istio-1-14/kubeflow-istio-resources/base | kubectl apply -f -
```

#### Kubeflow Pipelines

```Shell
./kustomize build apps/pipeline/upstream/env/cert-manager/platform-agnostic-multi-user | kubectl apply -f -
kubectl apply -f kubeflow-pv.yaml -l name=kubeflow
```

#### katib

```Shell
./kustomize build apps/katib/upstream/installs/katib-with-kubeflow | kubectl apply -f -
kubectl apply -f kubeflow-pv.yaml -l name=katib-mysql
```

#### Central Dashboard

```Shell
./kustomize build apps/centraldashboard/upstream/overlays/kserve | kubectl apply -f -
```

#### Admission Webhook

```Shell
./kustomize build apps/admission-webhook/upstream/overlays/cert-manager | kubectl apply -f -
```

#### Notebooks

```Shell
./kustomize build apps/jupyter/notebook-controller/upstream/overlays/kubeflow | kubectl apply -f -
```

#### Profiles + KFAM

```Shell
./kustomize build apps/profiles/upstream/overlays/kubeflow | kubectl apply -f -
```

#### Volumes Web App

```Shell
./kustomize build apps/volumes-web-app/upstream/overlays/istio | kubectl apply -f -
```

#### Tensorboard

```Shell
./kustomize build apps/tensorboard/tensorboards-web-app/upstream/overlays/istio | kubectl apply -f -
```

#### Training Operator

```Shell
./kustomize build apps/training-operator/upstream/overlays/kubeflow | kubectl apply -f -
```

#### User Namespace

```Shell
./kustomize build common/user-namespace/base | kubectl apply -f -
```
