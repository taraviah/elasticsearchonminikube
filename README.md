
# Elasticsearch on Minikube Kubernetes Cluster Installation

This is an installation guide for an Elasticsearch cluster of 1 master and 2 data nodes on Minikube.




## Install Minikube
Install minikube onto device (https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download) 

Linux
```bash
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```
MaxOs
```bash
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```
Windows

Run Powershell as administrator
```ps
New-Item -Path 'c:\' -Name 'minikube' -ItemType Directory -Force
Invoke-WebRequest -OutFile 'c:\minikube\minikube.exe' -Uri 'https://github.com/kubernetes/minikube/releases/latest/download/minikube-windows-amd64.exe' -UseBasicParsing

$oldPath = [Environment]::GetEnvironmentVariable('Path', [EnvironmentVariableTarget]::Machine)
if ($oldPath.Split(';') -inotcontains 'C:\minikube'){
  [Environment]::SetEnvironmentVariable('Path', $('{0};C:\minikube' -f $oldPath), [EnvironmentVariableTarget]::Machine)
}
```

## Bring up Minikube Cluster
After minikube is installed, create a minikube cluster
```bash
minikube start --nodes 4 --cpus 4 --memory 6144
```
Verify cluster nodes are all up and kube-system pods are all up
```bash
kubectl get nodes 
kubectl get pods -A
```
Enable storage-provisioner and default-storageclass addons
```bash
minikube addons enable default-storageclass

minikube addons enable storage-provisioner
```
## Install ECK Operator
Install CRDs (Custom Resource Definitions)
```bash
kubectl create -f crds.yaml
```
Install operator with RBAC rule
```bash
kubectl apply -f operator.yaml
```
Verify that operator is running
```bash
kubectl get all -n elastic-system
```

## Deploy an Elastic Cluster
Create a PersistentVolume for the elasticsearch-data
```bash
kubectl apply -f pv.yaml
```
Deploy Elasticsearch cluster
```bash
kubectl apply -f elasticsearch-cluster.yaml
```
Elasticsearch values yaml is configured with 1 master node and 3 worker/data nodes

Verify elasticsearch is in green health
```bash
kubectl get elasticsearch
```
## Connect to Elastic Cluster API
Get the http service and forward it
```bash
kubectl get services

kubectl port-forward service/elasticsearchcluster-es-http 9200
```
In new terminal , get secret for elasticsearch user and get password for elasticsearch
```bash
kubectl get secret | grep user

PASSWORD=$(kubectl get secret elasticsearchcluster-es-elastic-user -o=jsonpath='{.data.elastic}' | base64 --decode)
```
Access elastic api with password and port forwarding with curl, or with https://localhost:9200
```bash
curl -u "elastic:$PASSWORD" -k https://localhost:9200
```

## Deploy a Kibana Instance
Deploy kibana instance and check if resources are deployed
```bash
kubectl apply -f kibana.yaml
kubectl get all
kubectl get kibana
```

## Access Kibana and Elasticsearch
Get the kibana http service and port forward it

```bash
kubectl get services | grep kb
kubectl port-forward service/elasticsearchcluster-kb-http 5601
```

Get password as before
```bash
PASSWORD=$(kubectl get secret elasticsearchcluster-es-elastic-user -o=jsonpath='{.data.elastic}' | base64 --decode)
```
Access kibana api at https://localhost:5601

## Acknowledgements

 - [Elastic Guide](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-quickstart.html)
 - [Minikube](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download)
 - [Troubleshooting](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-troubleshooting-methods.html)

