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

7. ENEXA Platform: Deployment Workflow on Kubernetes
To deploy the ENEXA platform, you must first create the PersistentVolume (PV) and PersistentVolumeClaim (PVC) resources. Afterwards, apply the Deployment and Service manifests. For some resources, specific roles may be required—for details, please consult the latest documentation in the official Git repository.

Below is an example of the Kubernetes manifest files you can use as a starting point (please adjust these according to your environment and requirements):

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: enexa-shared-pv
spec:
  capacity:
    storage: 40Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs-storage
  nfs:
    server: [replace the IP ]
    path: /data/nfs/kubedata [if this path is not exist create or change] 

```

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  creationTimestamp: null
  labels:
    io.kompose.service: enexa-shared-dir-claim
  name: enexa-shared-dir-claim
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 40Gi
  storageClassName: nfs-storage

```

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: enexa-module-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Delete
  storageClassName: nfs-storage  # Updated to match the storage class for NFS
  nfs:
    server: [replace the IP ]
    path: /data/nfs/module  <<== here is the path which you copy the modules .ttl files 
```

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    creationTimestamp: null
    labels:
        io.kompose.service: enexa-module-dir-claim
    name: enexa-module-dir-claim
spec:
    accessModes:
        - ReadWriteMany
    resources:
        requests:
            storage: 1Gi
    storageClassName: nfs-storage

```


```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-creator
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["create", "get", "update", "delete", "list","watch"]
  - apiGroups: [""]
    resources: ["services"]
    verbs: ["create", "get", "update", "delete", "list", "watch"]

```

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
    name: pod-creator-binding
    namespace: default
subjects:
    - kind: ServiceAccount
      name: default  
      namespace: default
roleRef:
    kind: Role
    name: pod-creator
    apiGroup: rbac.authorization.k8s.io
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.26.0 (40646f47)
  creationTimestamp: null
  labels:
    io.kompose.service: enexa-service
  name: enexa-service
spec:
  replicas: 1
  selector:
    matchLabels:
      io.kompose.service: enexa-service
  strategy:
    type: Recreate
  template:
    metadata:
      annotations:
        kompose.cmd: kompose convert
        kompose.version: 1.26.0 (40646f47)
      creationTimestamp: null
      labels:
        io.kompose.service: enexa-service
    spec:
      containers:
        - name: enexa-service
          imagePullPolicy: IfNotPresent
          image: hub.cs.upb.de/enexa/images/enexa-service-dev:0.0.48 <<== use the latest veriosn 

          env: <<== these should update if need
            - name: ENEXA_MODULE_DIRECTORY
              value: /mnt/enexa-module-dir
            - name: ENEXA_SERVICE_URL
              value: http://enexa-service:8081/
            - name: ENEXA_SHARED_DIRECTORY
              value: /enexa
            - name: ENEXA_META_DATA_ENDPOINT <<== based on which triple store used 
              value: http://tentris-devwd-service:9080/sparql
#              value: http://fuseki-devwd-service:3030/enexa/
            - name: ENEXA_META_DATA_GRAPH
              value: http://example.org/meta-data
            - name: ENEXA_RESOURCE_NAMESPACE
              value: http://example.org/resource/
          ports:
            - containerPort: 8080
          resources: {}
          volumeMounts:
            - mountPath: /enexa
              name: enexa-shared-dir
            - mountPath: /mnt/enexa-module-dir
              name: enexa-module-dir
      restartPolicy: Always
      volumes:
        - name: enexa-shared-dir
          persistentVolumeClaim:
            claimName: enexa-shared-dir-claim
        - name: enexa-module-dir
          persistentVolumeClaim:
              claimName: enexa-module-dir-claim

```

```
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.26.0 (40646f47)
  creationTimestamp: null
  labels:
    io.kompose.service: enexa-service
  name: enexa-service
spec:
  ports:
    - name: "8081"  # Exposed port (can be any open port)
      port: 8081  # Target port (port the pod listens on)
      targetPort: 8081
  selector:
    io.kompose.service: enexa-service
  type: ClusterIP

```

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: fuseki-pv
spec:
  capacity:
    storage: 20Gi
  volumeMode: Filesystem  
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs-storage
  nfs:
    server: [use your NFS IP]
    path: /data/nfs/fusekidata

```

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  creationTimestamp: null
  labels:
    io.kompose.service: fuseki-pvc
  name: fuseki-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 20Gi
  storageClassName: nfs-storage
status: {}

```

```
apiVersion: apps/v1
kind: Deployment 
metadata:
    name: fuseki-devwd 
spec:
    replicas: 1
    selector:
        matchLabels:
            app: fuseki-devwd 
    template:
        metadata:
            labels:
                app: fuseki-devwd
        spec:
            imagePullSecrets:
                - name: my-dockerhub-secret
            containers:
                - name: fuseki-devwd
                  image: docker.io/stain/jena-fuseki
                  ports:
                      - containerPort: 3030
                        name: http # Optional name for the port
                  env:
                      - name: ADMIN_PASSWORD
                        value: [use password in ""] # Environment variable
                      - name: FUSEKI_DATASET_1
                        value: "enexa"
                      - name: FUSEKI_DATASET_2
                        value: "mydataset"   
                      - name: FUSEKI_ARGS
                        value: "--update --enableControl"
                  volumeMounts:
                      - name: fuseki-data # Volume name
                        mountPath: /fuseki # Path to mount volume in container
            volumes:
                - name: fuseki-data
                  persistentVolumeClaim:
                    claimName: fuseki-pvc

```

```
apiVersion: v1
kind: Service
metadata:
    name: fuseki-devwd-service
spec:
    selector:
        app: fuseki-devwd  # Match pods in the Deployment
    ports:
        - protocol: TCP
          port: 3030  # Port exposed by the Service
          targetPort: 3030  # Port on the pod to forward traffic to
          nodePort: 31001
    type: NodePort  # Default type, can be changed to NodePort or LoadBalancer if needed

```

DONE ! now the ENEXA platform deployed :)
