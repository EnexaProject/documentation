---
title: Quick Deployment Guide
keywords: ENEXA Documentation
sidebar: mydoc_sidebar
toc: false
permalink: quick_guide.html
folder: mydoc
---
## Setting Up Environment for Running the Service

To run the service, you need to prepare the environment. Below are the steps to do so:

1. **Prepare Metadata Directory:**
  - Create a directory named "metadata".
  - Place all required metadata files inside this directory.
  - For each module you require, you should place a `.ttl` file here. These `.ttl` files can be found in the module repository. If you are integrating your module with Enexa, please refer to this section ([Starting the Metadata File](https://enexa.eu/documentation/module_dev.html#2-starting-the-meta-data-file)) and add your module's `.ttl` file in the metadata directory.
  - For example, for a scenario, the contents of the metadata directory could be as follows:

   ```
   embeddings.ttl
   tentris.ttl
   cel-deploy.ttl
   extraction.ttl
   transform.ttl
   cel-train.ttl
   repair.ttl
   wikidata-preproc.ttl
   ```

2. **Create a Shared Directory:**
  - The entire platform requires a shared directory for exchanging files between containers. Since Docker is used, this directory will be utilized by Docker images as a mounted volume. It should be introduced to the service as a path with sufficient read and write access. To fulfill this requirement, create a directory and designate it as the shared directory. Set the path to this directory when running the service. 

3. **Run the triple store as META-DATA store**
   - Utilize any triple store of your preference, ensuring it offers an endpoint for executing SPARQL queries. One suggestion is to run Fuseki as a docker-compose, as outlined below:

    ``` 
    
    version: "3.0"
    services:
      fuseki:
        image: stain/jena-fuseki
        container_name: fuseki
        networks:
          - enexaNet
        ports:
          - "3030:3030"
        environment:
          ADMIN_PASSWORD: pw123
        volumes:
          - /data/fusekiData:/fuseki
    ```


## Running the Service

To run the Enexa service, Docker is required. First, ensure you have Docker installed by running the following command:

```
docker -v
```

If Docker is not installed, please install it before proceeding.

Next, pull the Docker image using one of the following commands:

```
docker pull hub.cs.upb.de/enexa/images/enexa-service-demo:1.2.0
or 
docker pull hub.cs.upb.de/enexa/images/enexa-service:latest (this image is not available yet)
```

After pulling the image, run the service using the following example Docker command. Replace all placeholders within brackets [] with valid data:

```
sudo docker run -d --name [set as you wish] --network [replace with network name] 
-v [replace with valid path on your machine]:/home/shared 
-v /var/run/docker.sock:/var/run/docker.sock 
-v [replace with valid path for metadata on your machine]:/home/metadata 
-e ENEXA_META_DATA_ENDPOINT=[triple store endpoint]  
-e ENEXA_META_DATA_GRAPH=[something like http://example.org/meta-data] 
-e ENEXA_MODULE_DIRECTORY=[set] 
-e ENEXA_RESOURCE_NAMESPACE=[set something like http://example.org/enexa/] 
-e ENEXA_SERVICE_URL=[on which URL this service would be available something like http://enexaservice:8080/] 
-e ENEXA_SHARED_DIRECTORY=[shared directory path] 
-e DOCKER_NET_NAME=[network name] 
[image name something like hub.cs.upb.de/enexa/images/enexa-service-demo:1.2.0]
```

Ensure all placeholders are replaced with the appropriate values according to your setup.


## Option Two: Running from the Code (Not Recommended - For Development Purposes Only)

- Clone the code from the Git repository [here](https://github.com/EnexaProject/enexa-service).
  - Set all these variables:
    ```
    export ENEXA_META_DATA_ENDPOINT=[triple store endpoint]
    export ENEXA_META_DATA_GRAPH=[something like http://example.org/meta-data]
    export ENEXA_MODULE_DIRECTORY=[set]
    export ENEXA_RESOURCE_NAMESPACE=[set something like http://example.org/enexa/]
    export ENEXA_SERVICE_URL=[on which URL this service would be available something like http://enexaservice:8080/]
    export ENEXA_SHARED_DIRECTORY=[shared directory path] 
    export DOCKER_NET_NAME=[network name] 
    ```

  - Use "mvn clean install".
  - Run it with java -jar [path of jar file like target/enexa-service-0.0.1-SNAPSHOT.jar].

## Testing Service is Up
- After running the service, you can call "[your api address like http://localhost:8080]/test". If you receive "OK", then the service is up.

# Deploying Enexa on Kubernetes


Enexa can also be deployed on a Kubernetes cluster. While you may use different environments, we recommend the following configuration for stability and compatibility:

- Kubernetes (kubeadm): v1.28.13  
- CRI-O: 1.28.4  
- OS: Debian GNU/Linux 11 (bullseye)  
- Cluster topology: At least 1 controller, 2 worker nodes, and an NFS server for shared storage

## 1. Prerequisites and Host Configuration

Ensure all nodes meet the minimum requirements and have correct host settings.

Disable Swap and Configure Kernel Modules

Swap must be disabled for kubelet to operate. The overlay and br_netfilter kernel modules must be loaded.

```
# Disable swap
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Load kernel modules and sysctl params
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
```

## 2. Install and Configure CRI-O

```
export OS=Debian_11
export VERSION=1.28
export CRIO_VERSION=1.28.4
echo "deb [signed-by=/usr/share/keyrings/libcontainers-archive-keyring.gpg] https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
echo "deb [signed-by=/usr/share/keyrings/libcontainers-crio-archive-keyring.gpg] https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo gpg --dearmor -o /usr/share/keyrings/libcontainers-archive-keyring.gpg
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/Release.key | sudo gpg --dearmor -o /usr/share/keyrings/libcontainers-crio-archive-keyring.gpg

sudo apt-get update
sudo apt-get install cri-o-cri-o=$CRIO_VERSION* -y
sudo apt-get install cri-o-cri-o-runc=$CRIO_VERSION* -y

sudo systemctl daemon-reload
sudo systemctl enable crio
sudo systemctl start crio
```

## 3. Install Kubernetes Components

```
KUBE_VERSION=1.28.13
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet=$KUBE_VERSION* kubeadm=$KUBE_VERSION* kubectl=$KUBE_VERSION*
sudo apt-mark hold kubelet kubeadm kubectl
```

4. Bootstrap the Cluster (Control Plane Only)

On the controller node:

```
sudo kubeadm init --cri-socket=unix:///var/run/crio/crio.sock --pod-network-cidr=192.168.0.0/16
```

After completion, follow the onscreen instructions to configure kubectl:

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

Install Calico CNI plugin for networking:

kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

Verify pods status:

```
kubectl get pods --all-namespaces
```

5. Join Worker Nodes to the Cluster

On each worker node, use the join command from kubeadm init (replace with your values):

```
sudo kubeadm join <controller-host>:<port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

Verify all nodes are ready from the controller:

kubectl get nodes

6. Create Docker Registry Secret

If your Docker images are in a private repository, create a Kubernetes secret for image pull:

kubectl create secret docker-registry my-dockerhub-secret \
  --docker-username=[your-username] \
  --docker-password=[your-password] \
  --docker-email=[your-email]

Once your cluster is running and all nodes are ready, you can proceed to deploy the Enexa platform.


