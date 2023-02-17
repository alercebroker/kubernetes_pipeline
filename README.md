# Kubernetes_pipeline
## Introduction


In this repository I will explain how to deploy the pipeline that currently runs docker-compose in a Kubernetes or K8s architecture.

## AWS EKS Cluster

Kubernestes as a platform, can be mounted locally with minikube or for development, staging and production environments in the AWS EKS service. This service provides agility for the creation of the cluster, horizontal scaling and operating times (uptime) better than having them mounted on an on-premise platform.

The cluster creation process would look something like this:

![](https://i.imgur.com/0sJ447h.png)
![](https://i.imgur.com/MpM8RUX.png)
![](https://i.imgur.com/Ltb46V1.png)
![](https://i.imgur.com/7dzcWWc.png)

## AWS EKS nodes

Once we have the cluster created, it is necessary to add the number of nodes that are necessary to support the workloads. Within the variables to be configured, the type of EC2 instance to occupy must be specified. The number of pods that we can have per node will depend on this.


In the following link we can take as a reference the number of pods per node depending on the type of instance:

https://github.com/awslabs/amazon-eks-ami/blob/master/files/eni-max-pods.txt

![](https://i.imgur.com/sL6cb2I.png)
![](https://i.imgur.com/KrYRYez.png)

## Kubectl


Kubectl is a command line tool for deploying and managing applications on Kubernetes. Using kubectl, we can inspect cluster resources, create, remove, and update components. 

### Installation ([Link](https://docs.aws.amazon.com/es_es/eks/latest/userguide/install-kubectl.html))
1) Download.
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
```

2) Execute permissions to the binary.
```
chmod +x ./kubectl
```

3) Copy the binary to a folder in your PATH.
````
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
````

4) Check your version with the following command.
````
kubectl version --short --client
````

5) Basic commands ([Link](https://kubernetes.io/docs/reference/kubectl/cheatsheet/))

```
# Get commands with basic output
kubectl get services                          # List all services in the namespace
kubectl get pods --all-namespaces             # List all pods in all namespaces
kubectl get pods -o wide                      # List all pods in the current namespace, with more details
kubectl get deployment my-dep                 # List a particular deployment
kubectl get pods                              # List all pods in the namespace
kubectl get pod my-pod -o yaml                # Get a pod's YAML

# Describe commands with verbose output
kubectl describe nodes my-node
kubectl describe pods my-pod

# List Services Sorted by Name
kubectl get services --sort-by=.metadata.name

# List pods Sorted by Restart Count
kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'

# List PersistentVolumes sorted by capacity
kubectl get pv --sort-by=.spec.capacity.storage

# Get the version label of all pods with label app=cassandra
kubectl get pods --selector=app=cassandra -o \
  jsonpath='{.items[*].metadata.labels.version}'

# Retrieve the value of a key with dots, e.g. 'ca.crt'
kubectl get configmap myconfig \
  -o jsonpath='{.data.ca\.crt}'

# Get all worker nodes (use a selector to exclude results that have a label
# named 'node-role.kubernetes.io/master')
kubectl get node --selector='!node-role.kubernetes.io/master'

# Get all running pods in the namespace
kubectl get pods --field-selector=status.phase=Running

# Get ExternalIPs of all nodes
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'

# List Names of Pods that belong to Particular RC
# "jq" command useful for transformations that are too complex for jsonpath, it can be found at https://stedolan.github.io/jq/
sel=${$(kubectl get rc my-rc --output=json | jq -j '.spec.selector | to_entries | .[] | "\(.key)=\(.value),"')%?}
echo $(kubectl get pods --selector=$sel --output=jsonpath={.items..metadata.name})

# Show labels for all pods (or any other Kubernetes object that supports labelling)
kubectl get pods --show-labels

# Check which nodes are ready
JSONPATH='{range .items[*]}{@.metadata.name}:{range @.status.conditions[*]}{@.type}={@.status};{end}{end}' \
 && kubectl get nodes -o jsonpath="$JSONPATH" | grep "Ready=True"

# List all Secrets currently in use by a pod
kubectl get pods -o json | jq '.items[].spec.containers[].env[]?.valueFrom.secretKeyRef.name' | grep -v null | sort | uniq

# List all containerIDs of initContainer of all pods
# Helpful when cleaning up stopped containers, while avoiding removal of initContainers.
kubectl get pods --all-namespaces -o jsonpath='{range .items[*].status.initContainerStatuses[*]}{.containerID}{"\n"}{end}' | cut -d/ -f3

# List Events sorted by timestamp
kubectl get events --sort-by=.metadata.creationTimestamp

# Compares the current state of the cluster against the state that the cluster would be in if the manifest was applied.
kubectl diff -f ./my-manifest.yaml

# Produce a period-delimited tree of all keys returned for nodes
# Helpful when locating a key within a complex nested JSON structure
kubectl get nodes -o json | jq -c 'path(..)|[.[]|tostring]|join(".")'

# Produce a period-delimited tree of all keys returned for pods, etc
kubectl get pods -o json | jq -c 'path(..)|[.[]|tostring]|join(".")'
```

## Access to private repositories

In our case, the images of each step are in a private repository, to be able to deploy the images we must make some settings that allow us to access. The steps are the following:

1) Have a git account that you have the permissions to access the packages.

2) Within the developer options, create a token with the read permissions to the packages.

3) The k8s recommendation is to save this token in a .txt and configure it as follows:
```
cat token.txt | docker login ghcr.io -u "user" --password-stdin
```

4) Log in to docker
````
docker login 
````

5) Test access
````
docker pull ghcr.io/alercebroker/xmatch_step:latest
````

6) Add configuration to the kubernetes cluster
````
kubectl create secret generic test \
    --from-file=.dockerconfigjson=/Users/youruser/.docker/config.json \
    --type=kubernetes.io/dockerconfigjson
````

7) Add secret configuration to manifest
````
    imagePullSecrets:
- name: test
````

## Manifest creation

As in docker-compose, in kubernetes, the applications deployments have been through manifests. These files have the necessary settings for everything to work. Its format must be .yaml .


Since these manifests already existed in docker-compose, we used the "Kompose" application to be able to transform the compose manifests to kubernetes manifests. 

Here is an example of the xmatch manifest:

````
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    kompose.cmd: kompose convert -f xmatch_step/docker-compose.yml
    kompose.version: 1.19.0 (f63a961c)
  creationTimestamp: null
  labels:
    io.kompose.service: xmatch
  name: xmatch
spec:
  replicas: 1
  selector:
    matchLabels:
      app: xmatch
  strategy: {}
  template:
    metadata:
      annotations:
        kompose.cmd: kompose convert -f xmatch_step/docker-compose.yml
        kompose.version: 1.19.0 (f63a961c)
      creationTimestamp: null
      labels:
        io.kompose.service: xmatch
        app: xmatch
    spec:
      containers:
      - env:
        - name: ES_HOST
          value: 10.0.2.126
        - name: LOGSTASH_USER
          value: metrics
        - name: LOGSTASH_PASSWORD
          value: xxxxxx
        - name: FILEBEAT_USER
          value: xxxxxx
        - name: FILEBEAT_PASSWORD
          value: alercemetrics
        - name: DB_ENGINE
          value: postgresql
        - name: DB_HOST
          value: 18.223.221.83
        - name: DB_USER
          value: k8s
        - name: DB_PASSWORD
          value: 'xxxxxx'
        - name: DB_PORT
          value: "5432"
        - name: DB_NAME
          value: new_pipeline_ts
        - name: CONSUMER_SERVER
          value: kafka1.alerce.online:9092,kafka2.alerce.online:9092,kafka3.alerce.online:9092
        - name: CONSUMER_TOPICS
          value: ztf_20201023_programid1_aux
        - name: CONSUMER_GROUP_ID
          value: xmatch_consumer_k8s
        - name: PRODUCER_SERVER
          value: 23.23.87.67:9092,35.174.222.219:9092,54.145.72.101:9092 #replace with private ips
        - name: PRODUCER_TOPIC
          value: xmatch_k8s
        - name: STEP_VERSION
          value: xmatch_test_k8s_0.0.1
        - name: METRICS_HOST
          value: 23.23.87.67:9092,35.174.222.219:9092,54.145.72.101:9092 #replace with private ips
        - name: METRICS_TOPIC
          value: metrics
        image: ghcr.io/alercebroker/xmatch_step
        name: xmatch
        resources: 
          limits: 
            cpu: "100m" #thousandth of a core
          requests: 
            cpu: "100m"
      restartPolicy: Always
      imagePullSecrets:
      - name: test
    # nodeSelector:
    #   disktype: xmatch
status: {}
````


## HPA horizontal pod autoscaler

Kubernetes has many apis that provide different services, within the most important is HPA (horizontal pod autoscaler).

HPA works by scaling pods in a step depending on how we configure it. The following example could be a use for the production environment:

* K8s cluster = 1
* Nodes = 6 (one per step)
* Pods min = 1
* Pods max = 10
* CPU = 70%


In this example we have one node per step, within each node the step will scale from 1 to 10 pods once the cpu exceeds 70%. This change is made dynamically depending on the workload.

## Monitoring 

By default we can monitor the cluster from kubectl using commands like "top" among others. But for productive environments, the use of tools such as rancher is recommended to be able to have notification services, real-time dashboard, etc.

For the deployment of the pipeline we do not use rancher, but when changing to the staging or production environment it will begin to be used.
(Our rancher server: Rancher - dev)

Referential image
![](https://i.imgur.com/kGgGMV7.png)


## Start up

TODO 
