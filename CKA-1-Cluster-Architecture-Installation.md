# 1. Cluster Architecture, Installation & Configuration 25%

Domain	| Weight
------- | -------------
[**1. Cluster Architecture, Installation & Configuration**](CKA-1-Cluster-Architecture-Installation.md#1-cluster-architecture-installation--configuration-25)	| 25%  
[1.1. Manage role based access control (RBAC)](CKA-1-Cluster-Architecture-Installation.md#11-manage-role-based-access-control-rbac) |  
[1.2. Use Kubeadm to install a basic cluster](CKA-1-Cluster-Architecture-Installation.md#12-use-kubeadm-to-install-a-basic-cluster) |  
[1.3. Manage a highly-available Kubernetes cluster](CKA-1-Cluster-Architecture-Installation.md#13-manage-a-highly-available-kubernetes-cluster) |  
[1.4. Provision underlying infrastructure to deploy a Kubernetes cluster](CKA-1-Cluster-Architecture-Installation.md#14-provision-underlying-infrastructure-to-deploy-a-kubernetes-cluster) |  
[1.5. Perform a version upgrade on a Kubernetes cluster using Kubeadm](CKA-1-Cluster-Architecture-Installation.md#15-perform-a-version-upgrade-on-a-kubernetes-cluster-using-kubeadm) |  
[1.6. Implement etcd backup and restore](CKA-1-Cluster-Architecture-Installation.md#16-implement-etcd-backup-and-restore) |

<br/><br/>

## 1.1. Manage Role-Based Access Control (RBAC)  
https://kubernetes.io/docs/reference/access-authn-authz/rbac/  

Role-based access control (RBAC) is a method of regulating access to resources based on the roles of individual users.

Objects: 
* **Role** (what permissions) <— **RoleBinding** (which users)  
    * in a namespace  
* **ClusterRole** <— **ClusterRoleBinding**  
    * in the whole cluster  
<br/>

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

To get documentation of various resources:
```
kubectl explain role --recursive
```  
<br/>

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
  
To get documentation of various resources:
```
kubectl explain rolebinding --recursive
```  

<br/><br/>

### Service Accounts
https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

Service Accounts are accounts used by container processes within Pods to authenticate and control access to K8s APIs.

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
<br/>
To get documentation of various resources:
```
kubectl explain sa --recursive
```  

<br/><br/>

## 1.2. Use Kubeadm to install a basic cluster  
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

Kubeadm is a tool to simplify building K8s clusters

Basics:
```
kubeadm init 
kubeadm join
kubeadm token create --print-join-command

#kubeadm reset  ### if init failed for some reason...
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

<br/><br/>

## 1.3. Manage a highly-available Kubernetes cluster  
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/  

### Control Plane HA   
Deploy K8s control plane on multiple nodes & ensure that a single node failure does not impact the cluster’s control plane functions.

- node#1: kube-api-server	
- node#2: kube-api-server
- node#3: kube-api-server
- node#4: kubelet
- node#5: kubelet
 ...

Load Balancer schedules tasks across redundant nodes 

<br/>

### Etcd Data Store  

- Stacked Etcd  
  - Etcd runs on each control plane nodes  
- External Etcd  
  - Etcd does not run on control plane nodes but on external nodes  
  
<br/><br/>

## 1.4. Provision underlying infrastructure to deploy a Kubernetes cluster
<br/>

### K8s Installation
[k8s-install-base-docker.sh](https://gist.github.com/tplisson/1bb67b45d4c92d83b22a6d1e20771234)  
[k8s-install-base-containerd.sh](https://gist.github.com/tplisson/caaf5ce57a95d6b3cd6af3d5b53aa15f)  
[Centos w kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)  

<br/>

### Install Container Runtime
Install Docker Engine  
https://docs.docker.com/engine/install/

Install Containerd  
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

<br/>

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
source <(kubectl completion bash)
alias k=kubectl
complete -F __start_kubectl k
```

<br/><br/>

## 1.5. Perform a version upgrade on a Kubernetes cluster using Kubeadm 
https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/  

Overview:
1. Upgrade Control Plane
    1. upgrade kubeadm
    2. drain node
    3. plan kubeadm upgrade
    4. apply kubeadm upgrade
    5. upgrade kubelet & kubectl
    6. uncordon node
2. Upgrade Worker node1
    1. upgrade kubeadm
    2. drain node
    3. upgrade kubeadm node
    4. upgrade kubelet & kubectl
    5. uncordon node
3. Upgrade Worker node2
    1. upgrade kubeadm
    2. drain node
    3. upgrade kubeadm node
    4. upgrade kubelet & kubectl
    5. uncordon node
4. … etc.

<br/>

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

<br/>

### Control Plane Nodes Upgrade

Upgrade Kubeadm to 1.20.2
```
sudo apt-mark unhold kubeadm && \
sudo apt-get update && \
sudo apt-get install -y kubeadm=1.20.2-00 && \
# or
sudo apt-get install -y --allow-change-held-packages kubeadm=1.20.2-00 && \
sudo apt-mark hold kubeadm && \
sudo kubeadm upgrade plan v1.20.2 && \
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
<br/>

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

<br/>

## 1.6. Implement Etcd backup and restore 
https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/  

[`etcd`](https://github.com/etcd-io/etcd) is a distributed reliable key-value store used as the backend data storage for K8s clusters. All k8s objects, applications and configurations are stored in etcd.

[`etcdctl`](https://github.com/etcd-io/etcd/tree/master/etcdctl) is a command line client for etcd.

Some simple commands:
```
etcdctl version
etcdctl member list --write-out=table
etcdctl endpoint status --write-out=table 
etcdctl put foo bar
etcdctl get foo
etcdctl get --from-key ''
```

Some flags are required:
* endpoints (gRPC endpoints)
* cacert
* cert
* key

**Note**: If using etcdctl versions earlier than v3.4, set ETCDCTL_API=3 to use v3 API.

These flags can be specified for each command:
```
etcdctl member list \
  --endpoints=10.0.1.101:2379 \
  --cacert=etcd-certs/etcd-ca.pem \
  --cert=etcd-certs/etcd-server.crt \
  --key=etcd-certs/etcd-server.key \
  --write-out=table \
  member list
+------------------+---------+--------+-------------------------+-------------------------+------------+
|        ID        | STATUS  |  NAME  |       PEER ADDRS        |      CLIENT ADDRS       | IS LEARNER |
+------------------+---------+--------+-------------------------+-------------------------+------------+
| becbde068816b65f | started | etcd-1 | https://10.0.1.101:2380 | https://10.0.1.101:2379 |      false |
+------------------+---------+--------+-------------------------+-------------------------+------------+
```

Global flags can also be set as environment variables:
```
export ETCDCTL_ENDPOINTS=10.0.1.101:2379
export ETCDCTL_CACERT=etcd-certs/etcd-ca.pem
export ETCDCTL_CERT=etcd-certs/etcd-server.crt
export ETCDCTL_KEY=etcd-certs/etcd-server.key
```

```
$ etcdctl member list --write-out=table 
+------------------+---------+--------+-------------------------+-------------------------+------------+
|        ID        | STATUS  |  NAME  |       PEER ADDRS        |      CLIENT ADDRS       | IS LEARNER |
+------------------+---------+--------+-------------------------+-------------------------+------------+
| becbde068816b65f | started | etcd-1 | https://10.0.1.101:2380 | https://10.0.1.101:2379 |      false |
+------------------+---------+--------+-------------------------+-------------------------+------------+
```

How to find out those values ? Locate the manifest file for ETCD that includes it:
```
find /etc/kubernetes/manifests/  
```
```
grep 2379 /etc/kubernetes/manifests/etcd.yaml  
grep etcd /etc/kubernetes/manifests/etcd.yaml  
```

Manifest Variable      | ETCDTL Flag    | Environment Variable | Sample Value  
---------------------- | -------------- | -------------------- | -----------  
`--listen-client-urls` | `--endpoints`  | `ETCDCTL_ENDPOINTS`  | `https://127.0.0.1:2379`
`--trusted-ca-file`    | `--cacert`     | `ETCDCTL_CACERT`     | `ca.crt`
`--cert-file`          | `--cert`       | `ETCDCTL_CERT`       | `server.crt`
`--key-file`           | `--key`        | `ETCDCTL_KEY`        | `server.key`
  
<br/>

### Backing up ETCD

Basics
```
etcdctl snapshot save <filename>
```

Example:
```
# Save a snapshot 
etcdctl snapshot save etcd_backup.db
  
# Verify the snapshot 
etcdctl snapshot status etcd_backup.db --write-out=table 
```

### Restoring an ETCD backup 

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
etcdctl snapshot restore etcd_backup.db

# Check etcd member list
etcdctl member list --write-out=table 

# Change ownership from root to etcd
sudo chown -R etcd:etcd /var/lib/etcd

# Start the etcd daemon
sudo systemctl start etcd

# Look up the value for the key cluster.name in the etcd cluster:
etcdctl get --from-key ''
```
