# Certified-Kubernetes-Associate-Prep
My Certified Kubernetes Associate (CKA) preparation notes.

#### CKA Curriculum 
https://www.cncf.io/certification/cka/  

Domain	| Weight
------- | -------------
[**1. Cluster Architecture, Installation & Configuration**](README.md#1-cluster-architecture-installation--configuration-25)	| 25%  
[1.1. Manage role based access control (RBAC)](README.md#11-manage-role-based-access-control-rbac) |  
[1.2. Use Kubeadm to install a basic cluster](README.md#11-use-kubeadm-to-install-a-basic-cluster) |  
[1.3. Manage a highly-available Kubernetes cluster](README.md#13-manage-a-highly-available-kubernetes-cluster) |  
[1.4. Provision underlying infrastructure to deploy a Kubernetes cluster](README.md#14-provision-underlying-infrastructure-to-deploy-a-kubernetes-cluster) |  
[1.5. Perform a version upgrade on a Kubernetes cluster using Kubeadm](README.md#15-perform-a-version-upgrade-on-a-kubernetes-cluster-using-kubeadm) |  
[1.6. Implement etcd backup and restore](README.md#16-implement-etcd-backup-and-restore) |  
[**2. Workloads & Scheduling**](README.md#2-workloads--scheduling-15)	| 15%  
[2.1. Understand deployments and how to perform rolling update and rollbacks](README.md#21-understand-deployments-and-how-to-perform-rolling-update-and-rollbacks) |  
[2.2. Use ConfigMaps and Secrets to configure applications](README.md#22-use-configmaps-and-secrets-to-configure-applications) |  
[2.3. Know how to scale applications](README.md#23-know-how-to-scale-applications) |  
[2.4. Understand the primitives used to create robust, self-healing, application deployments](README.md#24-understand-the-primitives-used-to-create-robust-self-healing-application-deployments) |  
[2.5. Understand how resource limits can affect Pod scheduling](README.md#25-understand-how-resource-limits-can-affect-pod-scheduling) |  
[2.6. Awareness of manifest management and common templating tools](README.md#26-awareness-of-manifest-management-and-common-templating-tools) |  
[**3. Services & Networking**]() 	| 20%  
[**4. Storage**]()	| 10%  
[**5. Troubleshooting**]()	| 30%  

2Hrs | Cost $300 | Online Exam
K8s version 1.20 (Jan 22, 2021)

[1. Cluster Architecture, Installation & Configuration 25%]()  
[2. K8s Installation](README.md#2-K8s-Installation)  
[3. K8s Upgrades](README.md#3-k8s-upgrades)  
[4. K8s High Availability](README.md#4-K8s-High-Availability)  
[5. Etcd Backup & Restore](README.md#5-etcd-backup--restore)  
[6. RBAC](README.md#6-RBAC-Role-Based-Access-Control)  
[7. Application Configurations](README.md#7-Application-Configurations)  


#### ACG Training 
https://learn.acloud.guru/course/certified-kubernetes-administrator/  
by William Boyd


## 1. Cluster Architecture, Installation & Configuration 25%

### 1.1. Manage Role-Based Access Control (RBAC)  
https://kubernetes.io/docs/reference/access-authn-authz/rbac/  

Role-based access control (RBAC) = a method of regulating access to resources based on the roles of individual users 

Objects: 
* **Role** (what permissions) <— **RoleBinding** (which users)  
    * in namespace  
* **ClusterRole** <— **ClusterRoleBinding**  
    * in whole cluster  

#### Role
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

#### RoleBinding
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

#### Service Accounts
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

### 1.2. Use Kubeadm to install a basic cluster

#### [Kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)  
Kubeadm = Tool to simplify building K8s clusters

Basics:
```
kubeadm init 
kubeadm join
kubeadm token create --print-join-command
kubeadm reset
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

### 1.3. Manage a highly-available Kubernetes cluster  
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/  

#### Control Plane HA   
Deploy K8s control plane on multiple nodes & ensure that a single node failure does not impact the cluster’s control plane functions.

node1: kube-api-server	
node2: kube-api-server
node3: kube-api-server
node4: kubelet
node5: kubelet
...

Load Balancer schedules tasks across redundant nodes 

#### Etcd Data Store  

- Stacked Etcd  
  - Etcd runs on each control plane nodes  
- External Etcd  
  - Etcd does not run on control plane nodes but on external nodes  
  

### 1.4. Provision underlying infrastructure to deploy a Kubernetes cluster

#### K8s Installation
[k8s-install-base-docker.sh](https://gist.github.com/tplisson/1bb67b45d4c92d83b22a6d1e20771234)  
[k8s-install-base-containerd.sh](https://gist.github.com/tplisson/caaf5ce57a95d6b3cd6af3d5b53aa15f) 
[Centos w kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) 

#### Install Container Runtime
Install Docker Engine  
https://docs.docker.com/engine/install/

Install Containerd  
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/


#### Management Tools
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

#### Kubectl autocomplete 
BASH
```
kubectl completion bash
```

### 1.5. Perform a version upgrade on a Kubernetes cluster using Kubeadm 
https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/  

#### [Draining a node](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)  
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

#### Control Plane Nodes Upgrade

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

#### Worker Nodes Upgrades

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

### 1.6. Implement Etcd backup and restore 
https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/  

#### Backing up Etcd

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

#### Restoring an Etcd backup 

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


## 2. Workloads & Scheduling 15%

### 2.1. Understand deployments and how to perform rolling update and rollbacks  
https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/

#### Deployments  
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
* Scaling up / down easily number of replicas
* Rolling updates to deploy new SW version

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-deployment
  template:
    metadata:
      labels:
        app: my-deployment
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.1
        ports:
        - containerPort: 80
```



#### Rolling Updates  
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment

3 options for Rolling Updates

1- Using imperative command “kubectl set image”
Use —record to easily rollback
```
kubectl set image deployment/my-deployment nginx=nginx:1.19.2 --record
```

2- Using imperative command “kubectl edit deploy”
```
kubectl edit deployment my-deployment
```
Change is applied immediately, no need for kubectl apply

3- Editing specs in the deployment YAML manifest 
```
kubectl apply -f deployment.yaml
```

Check status
```
kubectl rollout status deployment my-deployment
kubectl rollout status deployment/my-deployment
```

#### Rollback  
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment

3 options for Rolling back

1- Using kubectl set image
```
kubectl rollout history deployment/my-deployment
kubectl rollout undo deployment.v1.apps/my-deployment
kubectl rollout undo deployment.v1.apps/my-deployment --to-revision=1
```
Use "--to-revision" when "—-record" was used

2- Editing specs in the deployment YAML manifest 
```
kubectl apply -f deployment.yaml
```

3- Using kubectl edit
```
kubectl rollout status deployment my-deployment
```
change is applied immediately, no need for kubectl apply

Check status
```
kubectl rollout status deployment my-deployment
```


### 2.2. Use ConfigMaps and Secrets to configure applications

#### ConfigMaps 
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

#### Secrets 
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

#### Environment Variables 
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

#### Configuration Volumes  
https://kubernetes.io/docs/concepts/storage/volumes/

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webserver
spec:
  volumes: 
  - name: config-volume
    configMap:
      name: nginx-config
  - name: htpasswd-volume
    secret: 
      secretName: nginx-htpasswd
  containers:
  - name: webserver
    image: nginx:1.19.1
    ports:
    - containerPort: 80
    volumeMounts: 
    - name: config-volume
      mountPath: /etc/nginx
    - name: htpasswd-volume
      mountPath: /etc/nginx/conf
```

### 2.3. Know how to scale applications  
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment

3 options for Scaling

1- Using imperative command “kubectl scale”
```
kubectl scale deployment.v1.apps/my-deployment --replicas=5
```

2- Using imperative command “kubectl edit deploy”
```
kubectl edit deployment my-deployment
```
change is applied immediately, no need for kubectl apply

3- Editing replicas number in the deployment YAML manifest 
```
kubectl apply -f deployment.yaml
```
### 2.4. Understand the primitives used to create robust, self-healing, application deployments   
https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/
https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

#### Pod Probes  
##### Liveness Probes  
Testing whether the container is running

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-pod 
spec:
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'while true; do sleep 3600; done'] 
    livenessProbe: 
      exec:
        command: ["echo", "Hello, world!"]
      initialDelaySeconds: 5  ## Delay before kubelet triggers 1st probe
      periodSeconds: 5        ## How ofter kubelet performs a liveness probe
```

##### Startup Probes  
Testing whether the application within the container is started.
Useful for Pods that have containers that take a long time to come into service.
allowing a time longer than the liveness interval would allow.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-pod-http 
spec: 
  containers:
  - name: nginx 
    image: nginx:1.19.1 
    startupProbe: 
      httpGet:
        path: /
        port: 80
      failureThreshold: 30  ## Nb times try before restarting container
      periodSeconds: 10     ## How ofter kubelet performs a startup probe
```
##### Readiness Probes   
Testing whether a container is ready to start accepting traffic

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-pod 
spec: 
  containers:
  - name: nginx 
    image: nginx:1.19.1 
    readinessProbe: 
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5  ## Delay before kubelet triggers 1st probe
      periodSeconds: 5        ## How ofter kubelet performs probe
```

#### Restart policy  
https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy

##### Always  
For containers that should always run

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: always-pod 
spec:
  restartPolicy: Always  ## Default (so optional)
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'while true; do sleep 10; done']
```

##### OnFailure  
Restart ONLY IF the container exits w an error code or determined unhealthy with liveness probe

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: onfailure-pod 
spec:
  restartPolicy: OnFailure
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'while true; do sleep 10; done']
    command: ['sh', '-c', 'bad command that should fail my container']
```

##### Never  
For containers that should only be run once and never restarted.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: onfailure-pod 
spec:
  restartPolicy: Never 
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'while true; do sleep 10; done']
    command: ['sh', '-c', 'bad command that should fail my container']
```

#### Multi-Container Pods  
https://kubernetes.io/docs/concepts/workloads/pods/#using-pods
https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/
https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/

“Cross-container” interaction >> Network & storage
“Sidecar” = secondary container to the Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-pod 
spec:
  containers:
  - name: busybox1
    image: busybox
    command: ['sh', '-c', 'while true; do echo logs data > /output/output.log; sleep 5; done'] 
    volumeMounts:
    - name: sharedvol
      mountPath: /output
  - name: sidecar
    image: busybox
    command: ['sh', '-c', 'tail -f /input/output.log'] 
    volumeMounts:
    - name: sharedvol
      mountPath: /input
  volumes:
  - name: sharedvol 
    emptyDir: {}
```

##### Init-Container  
https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

Specialized containers that run before app containers in a Pod. 
Each init container must complete successfully before the next one starts.
If a Pod's init container fails, the kubelet repeatedly restarts that init container until it succeeds.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init1
    image: busybox:1.28
    command: ['sleep', '10']
  - name: init2
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup shipping-svc; do echo waiting for shipping-svc; sleep 2; done']
```


### 2.5. Understand how resource limits can affect Pod scheduling   
https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-requests-and-limits-of-pod-and-container

Resource Limits

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod 
spec: 
  containers:
  - name: busybox 
    image: busybox
    resources:
      requests:         ## Requests = estimate for scheduling
        memory: "64Mi"  ## in bytes, 64Mi = 64MiB
        cpu: "250m"     ## CPU units in 1/1000 of a CPU, 250m = 1/4
      limits:           ## Limits = enforced limit, stop if exceed
        memory: "128Mi"
        cpu: "500m"      
```

### 2.6. Awareness of manifest management and common templating tools   
https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/

* [helm](https://helm.sh)  
  * Templating (charts) & package mgmt  
* [kcompose](https://github.com/kubernetes/kompose)  
  * from Docker compose to K8s objects  
* [kustomize](https://kustomize.io)  
  * Config managment tool (similar to helm)  
  * https://kubernetes.io/blog/2018/05/29/introducing-kustomize-template-free-configuration-customization-for-kubernetes/ 

## 3. Services & Networking 20%
### 3.1. Understand host networking configuration on the cluster nodes
### 3.2. Understand connectivity between Pods
### 3.3. Understand ClusterIP, NodePort, LoadBalancer service types and endpoints 
### 3.4. Know how to use Ingress controllers and Ingress resources
### 3.5. Know how to configure and use CoreDNS
### 3.6. Choose an appropriate container network interface plugin 

## 4. Storage 10%
### 4.1. Understand storage classes, persistent volumes
### 4.2. Understand volume mode, access modes and reclaim policies for volumes
### 4.3. Understand persistent volume claims primitive
### 4.4 Know how to configure applications with persistent storage 

## 5. Troubleshooting 30%
### 5.1. Evaluate cluster and node logging
### 5.2. Understand how to monitor applications 
### 5.3. Manage container stdout & stderr logs
### 5.4. Troubleshoot application failure
### 5.5. Troubleshoot cluster component failure 
### 5.6 Troubleshoot networking 





