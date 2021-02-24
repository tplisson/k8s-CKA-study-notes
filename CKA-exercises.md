# CKA Exercises

[1. Cluster Architecture, Installation & Configuration - 25%](CKA-exercises.md#1-cluster-architecture-installation--configuration-25)  
[2. Workloads & Scheduling - 15%](CKA-exercises.md#2-workloads--scheduling-15)  



## 1. Cluster Architecture, Installation & Configuration**	| 25%  
## 1.1. Manage role based access control (RBAC)

Create a Pod able to get a list of Pods running in the default namespace.
You'll need to create a serviceAccount, a Role and a roleBinding, then use the following CURL command in your Pod:
```
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/default/pods/ --insecure
```
<details><summary>show solution</summary>
<p>

Create a serviceAccount:
```
kubectl create sa sa-demo
```
serviceAccount YAML file:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: sa-demo
  namespace: default
```
Create a Role allowing to list pods:
```
kubectl create role role-list-pods --verb=list --resource=pods
```
Role YAML file:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: role-list-pods
  namespace: default
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - list
```
Create a RoleBinding for the `role-list-pods` Role to the `sa-demo` serviceAccount:
```
kubectl create rolebinding rb-list-pods --serviceaccount=default:sa-demo --role=role-list-pods
```
RoleBinding YAML file:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rb-list-pods
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: role-list-pods
subjects:
- kind: ServiceAccount
  name: sa-demo
  namespace: default
```

Create a Pod using the `sa-demo` serviceAccount:
```
kubectl run -it pod-sa-demo --image=alpine --serviceaccount=sa-demo --rm
```

Pod YAML file:
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: pod-sa-demo
spec:
 serviceAccountName: sa-demo   #<---
 containers:
 - name: alpine
   image: alpine
   command:
   - "/bin/sh"
     - "-c"
     - "apk add curl"
     - "TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)"
     - 'curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/default/pods/ --insecure'
```

Verify you can get the list of Pods running in the default namespace from the API server:

```
apk add curl
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/default/pods/ --insecure
```

Pods list is shown: 
```
{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "710310"
  },
  "items": [
    ...
    
```
If it fails, you'd get this:
```
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {

  },
  "status": "Failure",
  "message": "pods is forbidden: User \"system:serviceaccount:default:default\" cannot list resource \"pods\" in API group \"\" in the namespace \"default\"",
  "reason": "Forbidden",
  "details": {
    "kind": "pods"
  },
  "code": 403
}
```

</p>
</details>
<br/>

## 1.5. Perform a version upgrade on a Kubernetes cluster using Kubeadm

Upgrade your cluster to Kubernetes version 1.20.2 using Kubeadm.

<details><summary>show solution</summary>
<p>

Upgrade the Control Plane:
```
kubeadm version 
sudo apt-mark unhold kubeadm 
sudo apt-get update 
sudo apt-get install -y kubeadm=1.20.2-00 
sudo apt-mark hold kubeadm
sudo kubeadm upgrade node 
kubeadm version 

kubectl drain <control-node> --ignore-daemonsets
kubectl version 
sudo apt-mark unhold kubelet kubectl 
sudo apt-get update 
sudo apt-get install -y kubelet=1.20.2-00 kubectl=1.20.2-00 
sudo apt-mark hold kubelet kubectl 
sudo systemctl daemon-reload 
sudo systemctl restart kubelet 
kubectl version

kubectl get nodes
```
Upgrade each Worker Node:
```
kubectl drain <worker-node> --ignore-daemonsets
ssh <worker-node>
sudo apt-get install -y --allow-change-held-packages kubeadm=<version>

sudo kubeadm upgrade node

sudo apt-get update && \
sudo apt-get install -y --allow-change-held-packages kubelet=<version> kubectl=<version>
sudo systemctl daemon-reload
sudo systemctl restart kubelet
exit
kubectl uncordon <control-node>
kubectl get nodes
```
</p>
</details>
<br/>

#### 2. Workloads & Scheduling	| 15%  
##### 2.1. Understand deployments and how to perform rolling update and rollbacks
##### 2.2. Use ConfigMaps and Secrets to configure applications
##### 2.3. Know how to scale applications

Excercise
<details><summary>show solution</summary>
<p>
```
```
</p>
</details>
<br/>

Excercise
<details><summary>show solution</summary>
<p>
```
```
</p>
</details>
<br/>