# Deploying Distributed MINIO S3 Server in minikube lab environment
## My lab
* minikube v1.23.0 with 4 CPUs and 4096 MB RAM using Docker VM Driver
* Host: Debian 11 Bullseye (VM) 4 CPUs and 8192 MB RAM
* 7 Sep 2021

Install minikube:
https://kubernetes.io/ru/docs/tasks/tools/install-minikube/

Install Dashboard:
https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

## Step-by-Step Guide
### 0. Clone this repo:
```git
git clone https://github.com/lepkov/minio-distributed-k8s
```
And move to downloaded dir 
### 1. Create NodePort
```
kubectl create -f 1NodePort.yaml
or
https://github.com/lepkov/minio-distributed-k8s/blob/main/1NodePort.yaml?raw=true
```
We need this service to forward 2 ports: 
- Application port: 9000 (App) to 31000 (Node)
- Console port: 32000 (App) to 32000 (Node) 

Console port is dynamic by default and supposed to be used with LoadBalancer in the Cloud, but in NodePort case it is necessary to make it static using argument in statefulset yaml file and forward it from 32000 (App) to 32000 (Node) 

### 2. Create HeadlessService
Headless Service controls the domain within which StatefulSets are created. The domain managed by this Service takes the form: $(service name).$(namespace).svc.cluster.local (where “cluster.local” is the cluster domain), and the pods in this domain take the form: $(pod-name-{i}).$(service name).$(namespace).svc.cluster.local. This is required to get a DNS resolvable URL for each of the pods created within the Statefulset. Create the Headless Service using the following command:
```
kubectl create -f 2HeadlessService.yaml
or
kubectl create -f https://github.com/lepkov/minio-distributed-k8s/blob/main/2HeadlessService.yaml?raw=true
```
### 3. Create StorageClass
```
kubectl create -f 3StorageClasss.yaml
or
kubectl create -f https://github.com/lepkov/minio-distributed-k8s/blob/main/3StorageClasss.yaml?raw=true
```
Here we specify that provisioner is minikube instead of AWS. And reclaimPolicy is set to Delete, this means that dynamically provisioned volume is automatically deleted when a user deletes the corresponding PersistentVolumeClaim.

### 4. Create StatefulSet with 4 replicas
```
kubectl create -f 4StatefulSet.yaml
or
kubectl create -f https://github.com/lepkov/minio-distributed-k8s/blob/main/4StatefulSet.yaml?raw=true
```
Here ports are binded as mentioned above. User and password are for demo purposes. Also servers are created using array 0,1,2,3. 
ReadWriteOnce means only 1 node can mount as read-write this volume. 
PVs will be created automatically, 4 items 1 Gigabyte each.

### 5. Finally, Distributed MINIO S3 Server is deployed on Kubernetes
```bash
curl $(minikube ip):32000
```
Open in modern browser http://(minikube ip):32000
![MINIO DEPLOYED ON KUBERNETES](https://github.com/lepkov/k8stasks/blob/main/Task3.2BonusTask.png)

## Helpful articles:
MINIO Quickstart Dockerhub
https://hub.docker.com/r/minio/minio

Deploy MinIO on Kubernetes
http://nm-muzi.com/docs/deploy-minio-on-kubernetes.html
