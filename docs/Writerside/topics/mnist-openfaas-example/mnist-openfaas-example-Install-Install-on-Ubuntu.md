# Install on Ubuntu

**Table of Contents**

- [Docker](#docker)
  - [Install Docker Engine on Ubuntu](#install-docker-engine-on-ubuntu)
    - [Step 1: Set up Docker's apt repository.](#step-1-set-up-docker-s-apt-repository)
    - [Step 2: Install the Docker packages.](#step-2-install-the-docker-packages)
- [OpenFaaS](#openfaas)
  - [Install on Kubernetes](#install-on-kubernetes)
    - [Step 1: Create Kubernetes Namespace](#step-1-create-kubernetes-namespace)
    - [Step 2: Helm charts](#step-2-helm-charts)
    - [(**Optional**) Step 3: Expose OpenFaaS Gateway Service to LoadBalancer](#optional-step-3-expose-openfaas-gateway-service-to-loadbalancer)
    - [Step 4: Get OpenFaaS admin Password](#step-4-get-openfaas-admin-password)
    - [Step 5: Login OpenFaaS WebUI via credential](#step-5-login-openfaas-webui-via-credential)
  - [OpenFaaS Command Line Tool (faas-cli)](#openfaas-command-line-tool-faas-cli)
  - [Build, Push, Deploy, Remove OpenFaaS Functions](#build-push-deploy-remove-openfaas-functions)
    - [Automatic (**Recommend**)](#automatic-recommend)
    - [Manual](#manual)
      - [Build OpenFaaS Functions](#build-openfaas-functions)
      - [Push OpenFaaS Functions](#push-openfaas-functions)
      - [Publish OpenFaaS Functions (Multi-Arch, **Recommend**)](#publish-openfaas-functions-multi-arch-recommend)
      - [Deploy OpenFaaS Functions](#deploy-openfaas-functions)
      - [Remove OpenFaaS Functions](#remove-openfaas-functions)
- [MinIO Object Storage for Linux](#minio-object-storage-for-linux)
  - [Server side](#server-side)
    - [Install MinIO Server](#install-minio-server)
    - [Launch MinIO Server](#launch-minio-server)
    - [Connect to MinIO WebUI via browser](#connect-to-minio-webui-via-browser)
  - [Client side](#client-side)
    - [Install MinIO Client](#install-minio-client)
    - [Configuration MinIO Client](#configuration-minio-client)
    - [Python Client API](#python-client-api)

## Docker

[Docker Official Documentation](https://docs.docker.com/engine/install/ubuntu/)

### Install Docker Engine on Ubuntu

#### Step 1: Set up Docker's apt repository.

```shell
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

#### Step 2: Install the Docker packages.

```shell
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## OpenFaaS

### Install on Kubernetes

#### Step 1: Create Kubernetes Namespace

```Shell
kubectl apply -f https://raw.githubusercontent.com/openfaas/faas-netes/master/namespaces.yml
```

#### Step 2: Helm charts

```Shell
helm repo add openfaas https://openfaas.github.io/faas-netes/
helm repo update
helm upgrade openfaas --install openfaas/openfaas --namespace openfaas
```

#### (**Optional**) Step 3: Expose OpenFaaS Gateway Service to LoadBalancer

```Shell
kubectl patch svc gateway-external -n openfaas -p '{"spec":{"type": "LoadBalancer"}}'
```

#### Step 4: Get OpenFaaS admin Password

```Shell
PASSWORD=$(kubectl -n openfaas get secret basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode) && \
echo "OpenFaaS admin password: $PASSWORD"
```

#### Step 5: Login OpenFaaS WebUI via credential

```Shell
# Default gateway endpoint is http://127.0.0.1:8080
faas-cli login -u admin -p $PASSWORD

# Specify gateway endpoint, for example http://10.0.0.196:31112
faas-cli login -u admin -p $PASSWORD -g http://10.0.0.196:31112
```

### OpenFaaS Command Line Tool (faas-cli)

<tabs>
<tab title="macOS">

```Shell
# via Homebrew
brew install faas-cli
```

</tab>
<tab title="Ubuntu">

```Shell
# root user
curl -sSL https://cli.openfaas.com | sudo -E sh

# non-root user
curl -sSL https://cli.openfaas.com | sh
```

</tab>
</tabs>

### Build, Push, Deploy, Remove OpenFaaS Functions

#### Automatic (**Recommend**)

```Shell
# default architecture is amd64
make faas-up

# if use multi-architecture then
make faas-up-multi-arch

# Specify gateway, for example Gateway is http://10.0.0.196:31112
make faas-up GATEWAY=http://10.0.0.196:31112
make faas-up-multi-arch GATEWAY=http://10.0.0.196:31112
```

#### Manual

##### Build OpenFaaS Functions

```Shell
make faas-build
```

##### Push OpenFaaS Functions

```Shell
make faas-push
```

##### Publish OpenFaaS Functions (Multi-Arch, **Recommend**)

publish = build + push

```Shell
# build and push multi-arch functions image (linux/arm64,linux/amd64)
make faas-publish
```

##### Deploy OpenFaaS Functions

```Shell
make faas-deploy

# Specify gateway, for example Gateway is http://10.0.0.196:31112
make faas-deploy GATEWAY=http://10.0.0.196:31112
```

##### Remove OpenFaaS Functions

```Shell
make faas-remove

# Specify gateway, for example Gateway is http://10.0.0.196:31112
make faas-remove GATEWAY=http://10.0.0.196:31112
```

## MinIO Object Storage for Linux

### Server side

[MinIO Official Documentation](https://min.io/docs/minio/linux/index.html)

#### Install MinIO Server

```Shell
cd ~
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
```

#### Launch MinIO Server

```Shell
cd ~
mkdir ~/minio-data
~/minio server ~/minio-data --console-address :9001
```

#### Connect to MinIO WebUI via browser

MinIO WebUI default account: `minioadmin`

MinIO WebUI default password: `minioadmin`

```
http://<HOST_IP>:9001

# Example
http://10.0.0.196:9001
```

### Client side

#### Install Minio Client

```Shell
cd ~
wget https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc
```

#### Configuration MinIO Client

```Shell
mc alias set <ALIAS_NAME> http://<HOST_IP>:9000 <ACCESS_KEY> <SECRET_KEY> 
mc admin info <ALIAS_NAME>

# Example
mc alias set local http://10.0.0.196:9000 minioadmin minioadmin
mc admin info local
```

#### Python Client API

```Python
from minio import Minio

def connect_minio():
    """Connect to MinIO Server"""

    MINIO_API_ENDPOINT = os.environ["minio_api_endpoint"]
    MINIO_ACCESS_KEY = os.environ["minio_access_key"]
    MINIO_SECRET_KEY = os.environ["minio_secret_key"]

    return Minio(
        MINIO_API_ENDPOINT,
        access_key=MINIO_ACCESS_KEY,
        secret_key=MINIO_SECRET_KEY,
        secure=False # Disable HTTPS
    )
```
