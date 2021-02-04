# Certified-Kubernetes-Associate-Prep
My Certified Kubernetes Associate (CKA) Preparation notes

## Contents
[1. CKA Details & Training resources](README.md#1-CKA-Details-&-Training-resources)  
[2. K8s Installation](README.md#2-K8s-Installation)  
[3. K8s Upgrades](README.md#3-k8s-upgrades)  
[4. K8s High Availability](README.md#4-K8s-High-Availability)  
[5. Etcd Backup & Restore](README.md#5-etcd-backup--restore)  
[6. RBAC](README.md#6-RBAC-Role-Based-Access-Control)  
[7. Application Configurations](README.md#7-Application-Configurations)  


## 1. CKA Details & Training resources

### CKA Curriculum 
https://www.cncf.io/certification/cka/  


Domain	| Weight
------- | -------------
Cluster Architecture, Installation & Configuration	| 25%
Workloads & Scheduling	| 15%
Services & Networking	| 20%
Storage	| 10%
Troubleshooting	| 30%

2Hrs | Cost $300 | Online Exam
K8s version 1.20 (Jan 22, 2021)


### ACG Training 
https://learn.acloud.guru/course/certified-kubernetes-administrator/  
by William Boyd



## 2. K8s Installation  
[k8s-install-base-docker-kube.sh](https://gist.github.com/tplisson/1bb67b45d4c92d83b22a6d1e20771234)  
[Centos w kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)  

### [Kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)  
Kubeadm = Tool to simplify building K8s clusters

Basics:
```
kubeadm init 
kubeadm join
kubeadm token create --print-join-command
```

Initialize K8s on the Control Plane
```
sudo kubeadm init --config <file.yml>
# or
sudo kubeadm init --pod-network-cidr=10.0.0.0/16

# Save the Join command or re-print it
kubeadm token create --print-join-command
```

Join the Cluster on Worker Nodes
```
sudo kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash <hash>
```

### Management Tools
https://kubernetes.io/docs/reference/tools/)  

* [kubectl](https://kubernetes.io/docs/reference/kubectl/)  
  * Main k8s CLI  
* [kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/)  
  * Clusters creation/deployment  
* [minikube](https://minikube.sigs.k8s.io/docs/start/)  
  * Quick single-node k8s cluster  
* [helm](https://helm.sh)  
  * Templating (charts) & package mgmt  
* [kcompose](https://github.com/kubernetes/kompose)  
  * from Docker compose to K8s objects  
* [kustomize](https://kustomize.io)  
  * Config managment tool (similar to helm)  
  * https://kubernetes.io/blog/2018/05/29/introducing-kustomize-template-free-configuration-customization-for-kubernetes/  


### Kubectl autocomplete 
BASH
```
kubectl completion bash
```

## 3. K8s Upgrades 
https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/  

### [Draining a node](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)  
To remove a node from service for upgrade or maintenance = gracefully terminate / move containers to other nodes

Drain a node
```
kubectl drain <node-name>
kubectl drain <node-name> --ignore-daemonsets # to avoid errors
```

After maintenance, allow pods to run on the node
```
kubectl uncordon <node-name>
```

### Control Plane Nodes Upgrade

Upgrade Kubeadm to 1.20.2
```
sudo apt-mark unhold kubeadm && \
sudo apt-get update && \
sudo apt-get install -y kubeadm=1.20.2-00 && \
# or
sudo apt-get install -y --allow-change-held-packages kubeadm=1.20.2-00 && \
sudo apt-mark hold kubeadm && \
sudo kubeadm upgrade apply v1.20.2 && \
kubeadm version
```

Upgrade Kubelet and Kubectl to 1.20.2
```
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && \
sudo apt-get install -y kubelet=1.20.2-00 kubectl=1.20.2-00 && \
sudo apt-mark hold kubelet kubectl && \
sudo systemctl daemon-reload && \
sudo systemctl restart kubelet
```

### Worker Nodes Upgrades

Upgrade Kubeadm to 1.20.2
```
sudo apt-mark unhold kubeadm && \
sudo apt-get update && \
sudo apt-get install -y kubeadm=1.20.2-00 && \
sudo apt-mark hold kubeadm && \
sudo kubeadm upgrade node && \
kubeadm version
```
Upgrade Kubelet and Kubectl to 1.20.2
```
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && \
sudo apt-get install -y kubelet=1.20.2-00 kubectl=1.20.2-00 && \
sudo apt-mark hold kubelet kubectl && \
sudo systemctl daemon-reload && \
sudo systemctl restart kubelet
```


## 4. K8s High Availability 
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/  

### Control Plane HA   
Deploy K8s control plane on multiple nodes & ensure that a single node failure does not impact the cluster’s control plane functions.

node1: kube-api-server	
node2: kube-api-server
node3: kube-api-server
node4: kubelet
node5: kubelet
...

Load Balancer schedules tasks across redundant nodes 

### Etcd Data Store  

- Stacked Etcd  
  - Etcd runs on each control plane nodes  
- External Etcd  
  - Etcd does not run on control plane nodes but on external nodes  
  


## 5. Etcd Backup & Restore
https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/  

### Backing up Etcd

Basics
```
etcdctl snapshot save <filename>
```

Example:
```
# Save a snapshot 
ETCDCTL_API=3 etcdctl snapshot save snapshot1 \
  --endpoints=https://[127.0.0.1]:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key 
  
# Verify the snapshot 
ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshot1
```

### Restoring an Etcd backup 

This creates a temporary logical cluster to repopulate data

Basics
```
etcdctl snapshot restore <filename>
```

Example
```
# Stop the etcd daemon
sudo systemctl stop etcd

# Delete existing etcd data
sudo rm -rf /var/lib/etcd

# Restore a backup
etcdctl snapshot restore <filename>

# Check etcd member list
ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://[127.0.0.1]:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key 

# Change ownership from root to etcd
sudo chown -R etcd:etcd /var/lib/etcd

# Start the etcd daemon
sudo systemctl start etcd

# Look up the value for the key cluster.name in the etcd cluster:
ETCDCTL_API=3 etcdctl get cluster.name \
  --endpoints=https://[127.0.0.1]:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key 
```


## 6. RBAC - Role-Based Access Control
https://kubernetes.io/docs/reference/access-authn-authz/rbac/  

Role-based access control (RBAC) = a method of regulating access to resources based on the roles of individual users 

Objects: 
* **Role** (what permissions) <— **RoleBinding** (which users)  
    * in namespace  
* **ClusterRole** <— **ClusterRoleBinding**  
    * in whole cluster  

### Role
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

### RoleBinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: User
  name: jane # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role | ClusterRole
  kind: Role       # Role | ClusterRole
  name: pod-reader # must match name of Role | ClusterRole to bind to
  apiGroup: rbac.authorization.k8s.io
```

### Service Accounts
https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

Service Account = account used by container processes within pods to authenticate and control access to K8s APIs

```yaml
Service Account
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-sa
  namespace: custom
```
or using imperative command:
```
kubectl create sa <sa-name> -n <namespace>
```

RoleBinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sa-read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: my-sa
  namespace: custom
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```


## 7. Application Configurations  

### ConfigMaps 
https://kubernetes.io/docs/concepts/configuration/configmap/  
ConfigMap = API object used to store non-confidential data in key-value pairs. 
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  # property-like keys; each key maps to a simple value
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # file-like keys
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5    
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true  
```

### Secrets 
https://kubernetes.io/docs/concepts/configuration/secret/  
Secrets = API object used to store and manage sensitive information, such as passwords, OAuth tokens, and SSH keys

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: opaque       # opaque = arbitrary user-defined data
data:
  secretkey1: <base64-string1>
```
Or using the imperative command
```
kubectl create secret generic my-secret --from-file=path/to/bar
```

How to get a base64 key from a string:
```
echo -n 'secret' | base64 
```

### Environment Variables 
https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/  

```yaml
apiVersion: v1
kind: Pod
...
spec:
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'echo "configmap: $CONFIGMAPVAR secret: $SECRETVAR"'] 
    env:
    - name: CONFIGMAPVAR
      valueFrom:
        configMapKeyRef:
          name: my-configmap
          key: key1
    - name: SECRETVAR
      valueFrom:
        secretKeyRef:
          name: my-secret 
          key: secretkey1
```
