# CKA Lab Exercises

Domain	| Weight
------- | -------------
[1. Cluster Architecture, Installation & Configuration](CKA-exercises.md#1-cluster-architecture-installation--configuration-25)  |  25%  
[2. Workloads & Scheduling](CKA-exercises.md#2-workloads--scheduling-15)  |  15%  



## 1. Cluster Architecture, Installation & Configuration**	| 25%  
### 1.1. Manage role based access control (RBAC)

#### Excercise 1  
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
    - "sleep 1000"
```

Verify you can get the list of Pods running in the default namespace from the API server:

```
kubectl exec pod-sa-demo -it -- sh
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

#### Excercise 2  
Create another Pod able to list any Pod running in any namespace.
You can use the following CURL command in your Pod (replace the <non-default> variable with one of the namespaces deployed in your cluster):
```
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/default/pods/ --insecure
curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/<non-default>/pods/ --insecure
```
<details><summary>show solution</summary>
<p>

Create a serviceAccount:
```
kubectl create sa sa-demo2
```
serviceAccount YAML file:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: sa-demo2
  namespace: default
```
Create a ClusterRole allowing to list pods:
```
kubectl create clusterrole cr-list-pods --verb=list --resource=pods
```
Role YAML file:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cr-list-pods
  namespace: default
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - list
```
Create a ClusterRoleBinding for the `cr-list-pods` ClusterRole to the `sa-demo2` serviceAccount:
```
kubectl create clusterrolebinding crb-list-pods --serviceaccount=default:sa-demo2 --clusterrole=cr-list-pods
```
RoleBinding YAML file:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: crb-list-pods
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cr-list-pods
subjects:
- kind: ServiceAccount
  name: sa-demo2
  namespace: default
```

Create a Pod using the `sa-demo2` serviceAccount:
```
kubectl run -it pod-sa-demo2 --image=alpine --serviceaccount=sa-demo2 --rm
```

Pod YAML file:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-sa-demo2
spec:
  serviceAccountName: sa-demo2   #<---
  containers:
  - name: alpine
    image: alpine
    command:
    - "/bin/sh"
    - "-c"
    - "sleep 1000"
```

Verify that you can get the list of Pods running in the any namespace from the API server:
```
kubectl exec pod-sa-demo2 -it -- sh
apk add curl
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/default/pods/ --insecure
curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/ns1/pods/ --insecure
curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/ns2/pods/ --insecure
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


### 1.5. Perform a version upgrade on a Kubernetes cluster using Kubeadm

#### Excercise 1  
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

## 2. Workloads & Scheduling	| 15%  
### 2.1. Understand deployments and how to perform rolling update and rollbacks

#### Excercise 1 - Create and update an NGINX deployment

1. Create a deployment with 3 pods, each containing a single nginx:1.19.6 container
2. Identify the update strategy employed by this deployment
3. Modify the update strategy so maxSurge is equal to 50% and maxUnavailable is equal to 50%
4. Scale the deployment to 5 pods
5. Perform a rolling update to this deployment so that the image gets updated to nginx1.19.7
6. Undo the most recent change

<details><summary>show solution</summary>
<p>

1. Create a deployment with 3 pods, each containing a single nginx:1.19.6 container:

```
kubectl create deploy nginx --image=nginx:1.19.6 --replicas=3
```
Deployment YAML file:
```yaml
apiVersion: v1
items:
- apiVersion: apps/v1
  kind: Deployment
  metadata:
    labels:
      app: nginx
    name: nginx
    namespace: default
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx:1.19.6      
```
Verify:
```
kubectl get deploy nginx -o wide
```

2. Identify the update strategy employed by this deployment:
```
kubectl describe deploy nginx | grep Strategy
```

3. Modify the update strategy so maxSurge is equal to 50% and maxUnavailable is equal to 50%

Edit the deployment API resource directly:
```
kubectl edit deploy nginx
```
Or create a patch file for the new rollingUpdate values and use `kubectl patch`:
```
vi patch.yml
```
```yaml
  spec:
    strategy:
      rollingUpdate:
        maxSurge: 50%
        maxUnavailable: 50%
```

```
kubectl patch deploy nginx --patch-file=patch.yml

```
Verify:
```
kubectl describe deploy nginx | grep Strategy
```

4. Scale the deployment to 5 pods

```
kubectl scale deploy nginx --replicas=5 --record
```
Or edit the deployment API resource directly:
```
kubectl edit deploy nginx
```

Verify:
```
kubectl get deploy nginx -o wide
```

5. Perform a rolling update to this deployment so that the image gets updated to nginx1.19.7

```
kubectl set image deploy nginx nginx=nginx:1.19.7 --record
```
Verify:
```
kubectl get deploy nginx -o wide
kubectl get po -o wide
```

6. Undo the most recent change

```
kubectl rollout history deployment nginx
kubectl rollout undo deployment nginx
```
Verify:
```
kubectl rollout history deployment nginx
kubectl get deploy nginx -o wide
```

</p>
</details>
<br/>


### 2.2. Use ConfigMaps and Secrets to configure applications

#### Excercise 1 - 
<details><summary>show solution</summary>
<p>
```
```
</p>
</details>
<br/>

#### Excercise 2 - 
<details><summary>show solution</summary>
<p>
```
```
</p>
</details>
<br/>


### 2.3. Know how to scale applications

#### Excercise 1 - 
<details><summary>show solution</summary>
<p>
```
```
</p>
</details>
<br/>

#### Excercise 2 - 
<details><summary>show solution</summary>
<p>
```
```
</p>
</details>
<br/>


### 2.4. Understand the primitives used to create robust, self-healing, application deployments	

#### Excercise 1 - 
<details><summary>show solution</summary>
<p>
```
```
</p>
</details>
<br/>

#### Excercise 2 - 
<details><summary>show solution</summary>
<p>
```
```
</p>
</details>
<br/>

### 2.5. Understand how resource limits can affect Pod scheduling	

#### Excercise 1 - 
<details><summary>show solution</summary>
<p>
```
```
</p>
</details>
<br/>

#### Excercise 2 - 
<details><summary>show solution</summary>
<p>
```
```
</p>
</details>
<br/>

###  2.6. Awareness of manifest management and common templating tools

#### Excercise 1 - 
<details><summary>show solution</summary>
<p>
```
```
</p>
</details>
<br/>

#### Excercise 2 - 
<details><summary>show solution</summary>
<p>
```
```
</p>
</details>
<br/>
