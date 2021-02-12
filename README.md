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
[**3. Services & Networking**](README.md#3-services--networking-20) 	| 20%  
[3.1. Understand host networking configuration on the cluster nodes](README.md#31-understand-host-networking-configuration-on-the-cluster-nodes)  |  
[3.2. Understand connectivity between Pods](README.md#32-understand-connectivity-between-pods) |  
[3.3. Understand ClusterIP, NodePort, LoadBalancer service types and endpoints](README.md#33-understand-clusterip-nodeport-loadbalancer-service-types-and-endpoints) |  
[3.4. Know how to use Ingress controllers and Ingress resources](README.md#34-know-how-to-use-ingress-controllers-and-ingress-resources) |  
[3.5. Know how to configure and use CoreDNS](README.md#35-know-how-to-configure-and-use-coredns) |  
[3.6. Choose an appropriate container network interface plugin](README.md#36-choose-an-appropriate-container-network-interface-plugin) |  
[**4. Storage**](README.md#4-storage-10)	| 10%  
[4.1. Understand storage classes, persistent volumes](README.md#41-understand-storage-classes-persistent-volumes) |  
[4.2. Understand volume mode, access modes and reclaim policies for volumes](README.md#42-understand-volume-mode-access-modes-and-reclaim-policies-for-volumes) |  
[4.3. Understand persistent volume claims primitive](README.md#43-understand-persistent-volume-claims-primitive) |  
[4.4 Know how to configure applications with persistent storage](README.md#44-know-how-to-configure-applications-with-persistent-storage) |   
[**5. Troubleshooting**]()	| 30%  
[5.1. Evaluate cluster and node logging](README.md#51-evaluate-cluster-and-node-logging) |  
[5.2. Understand how to monitor applications](README.md#52-understand-how-to-monitor-applications) |  
[5.3. Manage container stdout & stderr logs](README.md#53-manage-container-stdout--stderr-logs) |  
[5.4. Troubleshoot application failure](README.md#54-troubleshoot-application-failure) |  
[5.5. Troubleshoot cluster component failure](README.md#55-troubleshoot-cluster-component-failure) |  
[5.6 Troubleshoot networking](README.md#56-troubleshoot-networking) |  

2Hrs | Cost $300 | [Online Exam](https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/)
K8s version 1.20 (Jan 22, 2021)
 

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
source <(kubectl completion bash)
alias k=kubectl
complete -F __start_kubectl k
```



### 1.5. Perform a version upgrade on a Kubernetes cluster using Kubeadm 
https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/  

Overview:
1. Upgrade Control Plane
    1. upgrade kubeadm
    2. drain node
    3. upgrade kubelet & kubectl
    4. uncordon node
2. Upgrade Worker node1
    1. upgrade kubeadm
    2. drain node
    3. upgrade kubelet & kubectl
    4. uncordon node
3. Upgrade Worker node2
    1. upgrade kubeadm
    2. drain node
    3. upgrade kubelet & kubectl
    4. uncordon node
4. … etc.


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

Three options for rolling updates

1- Using imperative command “kubectl set image”
Use —record to easily rollback
```
kubectl set image deployment/my-deployment nginx=nginx:1.19.2 --record
```

2- Using imperative command “kubectl edit deploy”
```
kubectl edit deployment my-deployment
```
Any change is applied immediately, no need for kubectl apply

3- Editing specs in the deployment YAML manifest 
```
vi deployment.yaml
kubectl apply -f deployment.yaml
```

Check status
```
kubectl rollout status deployment my-deployment
kubectl rollout status deployment/my-deployment
kubectl rollout history deployment my-deployment
```

#### Rollback  
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment

Three options for rolling back

1- Using imperative command “kubectl rollout undo”
```
kubectl rollout history deployment/my-deployment
kubectl rollout undo deployment.v1.apps/my-deployment
kubectl rollout undo deployment.v1.apps/my-deployment --to-revision=1
```
Use "--to-revision" when "—-record" was used


2- Using imperative command “kubectl edit deploy”
```
kubectl edit deployment my-deployment
kubectl rollout status deployment my-deployment
```
Any change is applied immediately, no need for kubectl apply


3- Editing specs in the deployment YAML manifest 
```
vi deployment.yaml
kubectl apply -f deployment.yaml
```


Check status
```
kubectl rollout status deployment my-deployment
kubectl rollout history deployment my-deployment
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

Three options for scaling

1- Using imperative command “kubectl scale”
```
kubectl scale deployment.v1.apps/my-deployment --replicas=5
```


2- Using imperative command “kubectl edit deploy”
```
kubectl edit deployment my-deployment
```
Any change is applied immediately, no need for kubectl apply


3- Editing replicas number in the deployment YAML manifest 
```
kubectl apply -f deployment.yaml
```



### 2.4. Understand the primitives used to create robust, self-healing, application deployments   
https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/
https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

#### Liveness Probes  
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

#### Startup Probes  
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

#### Readiness Probes   
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
  
  
#### Resource Limits  

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
  
  
#### K8s Scheduling  
https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/  
https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/  

Using nodeName = only run Pod on this specific node
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  nodeName: node1
```

Using nodeSelector = run Pod on any node with matching label(s)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    myLabel1: mustMatchString
    myLabel2: "true"
    disk: fast
```

Label included in Node definition
```yaml
apiVersion: v1
kind: Node
metadata:
  labels:
    disk: fast
```


#### DaemonSets  
https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/  
A DaemonSet ensures that all (or some) Nodes run a copy of a Pod.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      name: nginx
  template:
    metadata:
      labels:
        name: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```


#### Static Pods   
https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/  
Pod managed directly by Kubelet, not by K8s API server

Mirror Pod created to represent a Static Pod in the Kuberntes API, allowing you to easily view the Static Pod's status. But no change can be done to them via API. 

Manifest path for static pods on each node under:
```
/etc/kubernetes/manifests/
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
https://kubernetes.io/docs/concepts/cluster-administration/networking/

Each Pod has its own unique IP address within the cluster 
>> 1 virtual network allowing by default all pods to communicate with each other

### 3.2. Understand connectivity between Pods  
By default, pods are non-isolated; they accept traffic from any source.

#### NetworkPolicy  
https://kubernetes.io/docs/concepts/services-networking/network-policies/  
If any networkPolicy selects a Pod, then the Pod is completely isolated and will only be open to traffic allowed by networkPolicy.

* podSelector = which pod to apply the networkPolicy
* ingress = into the Pod
* egress = leaving the Pod

##### Default deny all ingress traffic 
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

##### Default allow all ingress traffic 
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - {} #<-- means all
```

##### Sample Network Policy
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:     # or podSelector | namespaceSelector
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:               # Only this is allowed
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 597
```


### 3.3. Understand ClusterIP, NodePort, LoadBalancer service types and endpoints  
https://kubernetes.io/docs/concepts/services-networking/service/

Service = expose apps running as a set of Pods 
Service routing = load balancing from Client to Pods
Endpoints = Pods servicing the apps

Service Types:
* ClusterIP = expose apps inside cluster
* NodePort = expose apps outside cluster
* LoadBalancer = expose apps outside using external load balancer (eg. AWS ALB…)
* ExternalName (CKA out of scope)

#### ClusterIP
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-clusterip
  namespace: default
spec:
  type: ClusterIP
  selector:
    app: my-deployment
  ports:
  - port: 80        # service is listening to this port within cluster
    protocol: TCP
    targetPort: 80  # pods are listening to this port
```

#### NodePort
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-clusterip
  namespace: default
spec:
  type: NodePort
  selector:
    app: my-deployment
  ports:
  - port: 80        # service is listening to this port within cluster
    protocol: TCP
    targetPort: 80  # pods are listening to this port
    nodePort: 30080 # all nodes listening to this port 
```

#### DNS for Services and Pods   
https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/

Each Pod automatically gets a DNS name:
```
<pod-ip-addr>.<namespace>.pod.cluster.local
192-168-100-1.default.pod.cluster.local
```

Each Service automatically gets a DNS name:
```
<service-name>.<namespace>.svc.<cluster-domain>
<service-name>.default.svc.cluster.local
```

DNS names can be used to access a service from any namespace



### 3.4. Know how to use Ingress controllers and Ingress resources  
https://kubernetes.io/docs/concepts/services-networking/ingress/
https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/

Ingress manages external access to the services in a cluster, typically HTTP.
Ingress may provide load balancing, SSL termination and name-based virtual hosting.

Requires an Ingress Controller

Routing Rules with a set of Paths, each with a Backend >> Service

If Service uses a Named Port, then Ingress can refer to the Named Port (‘web’) instead of number (’80’)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: 80
```


### 3.5. Know how to configure and use CoreDNS  
https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/

Check that CoreDNS is running on control plane
```
kubectl get deployment kube-dns --n kube-system 
```

Each Pod is automatically assigned a DNS name:
```
<pod-ip-addr-w-dashes>.<namespace>.pod.cluster.local
192-168-100-1.default.pod.cluster.local
```
Each Service is automatically assigned a DNS name:
```
<service-name>.<namespace>.svc.<cluster-domain>
svc-frontend.default.svc.cluster.local
```


### 3.6. Choose an appropriate container network interface plugin  
https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/

K8s nodes will remain “NotReady” until a network plugin is installed. 

Each CNI plugin has its own implementation and own installation procedure. 

#### Calico
Installing Calico
```
kubectl apply -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml
```



## 4. Storage 10%  

### 4.1. Understand storage classes, persistent volumes  

#### StorageClass object  
https://kubernetes.io/docs/concepts/storage/storage-classes/  

Defining your class of Storage
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-localdisk
provisioner: kubernetes.io/no-provisioner # <--- no | AWS EBS | AzureFile | CephFS | FS | Local …
allowVolumeExpansion: true                # <---- a PVC can only expand if this is set to true 
```


#### Volumes  
https://kubernetes.io/docs/concepts/storage/volumes/  
https://kubernetes.io/docs/concepts/storage/persistent-volumes/  

Container File Sytems = ephemeral !  
Volumes allow Persistant Storage  

Volume type:
* emptyDir = dynamically created, useful to share data between containers (ephemeral)
* hostPath = directory on local node
* NFS
* Cloud Storage
* ConfigMaps & Secrets
* Simple Directory on K8s node

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: host-volume-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'while true; do echo "Success!" >> /output/success.txt; sleep 5; done']
    volumeMounts:
    - name: my-volume
      mountPath: /output
  volumes:
  - name: my-volume
    hostPath:
      path: /home/cloud_user/
      type: Directory

---
apiVersion: v1
kind: Pod
metadata:
  name: shared-volume-pod
spec:
  containers:
  - name: busybox1
    image: busybox
    command: ['sh', '-c', 'while true; do echo "Success!" >> /output/success.txt; sleep 5; done']
    volumeMounts: 
    - name: my-volume
      mountPath: /output
  - name: busybox2
    image: busybox
    command: ['sh', '-c', 'while true; cat /input/success.txt; sleep 5; done']
    volumeMounts: 
    - name: my-volume
      mountPath: /input
  volumes: 
  - name: my-volume
    emptyDir: {}
```


#### Persistent Volumes  
https://kubernetes.io/docs/concepts/storage/persistent-volumes/  
* K8s Object (not simply referred to in a Pod Object)
* More advanced form of Volume. 
* Allow to treat Storage as an abstract resource and consume it using Pods.

PersistentVolume (PV) object  
https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  storageClassName: my-localdisk
  persistentVolumeReclaimPolicy: Recycle
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /var/output

#persistentVolumeReclaimPolicy = Recycle | Retain | Delete
```

#### Types of Persistent Volumes

Supported PV plugins | Description
-------------------- | -------------
awsElasticBlockStore | AWS Elastic Block Store (EBS)
azureDisk | Azure Disk
azureFile | Azure File
cephfs | CephFS volume
cinder | Cinder (OpenStack block storage) (deprecated)
csi | Container Storage Interface (CSI)
fc | Fibre Channel (FC) storage
flexVolume | FlexVolume
flocker | Flocker storage
gcePersistentDisk | GCE Persistent Disk
glusterfs | Glusterfs volume
hostPath | HostPath volume (for single node testing only; WILL NOT WORK in a multi-node cluster; consider using local volume instead)
iscsi | iSCSI (SCSI over IP) storage
local | local storage devices mounted on nodes.
nfs | Network File System (NFS) storage
photonPersistentDisk | Photon controller persistent disk. (This volume type no longer works since the removal of the corresponding cloud provider.)
portworxVolume | Portworx volume
quobyte | Quobyte volume
rbd - Rados Block Device (RBD) volume
scaleIO | ScaleIO volume (deprecated)
storageos | StorageOS volume
vsphereVolume | vSphere VMDK volume




### 4.2. Understand volume mode, access modes and reclaim policies for volumes  

#### volume mode  
There are two ways PVs may be provisioned: statically or dynamically.


#### Access mode   
https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes

The access modes are:
* ReadWriteOnce -- the volume can be mounted as read-write by a single node
* ReadOnlyMany -- the volume can be mounted read-only by many nodes
* ReadWriteMany -- the volume can be mounted as read-write by many nodes

```yaml
apiVersion: v1
kind: PersistentVolume
spec:
  accessModes:
    - ReadWriteOnce
  ...
```

In the CLI, the access modes are abbreviated to:

RWO - ReadWriteOnce
ROX - ReadOnlyMany
RWX - ReadWriteMany


#### Reclaim Policies   
https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reclaiming

Reclaim Policies:
* Recycle 
* Retain
* Delete

```yaml
apiVersion: v1
kind: PersistentVolume
spec:
  ...
  persistentVolumeReclaimPolicy: Recycle
  ...
```



### 4.3. Understand persistent volume claims primitive  
https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reserving-a-persistentvolume  
object = user’s request w attributes 

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: my-localdisk
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
```

“Expand” a PVC = edit spec.resource.request.storage attribute of existing PVC
(must support allowVolumeExpansion = true)



### 4.4 Know how to configure applications with persistent storage   

Using PVC in a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pv-pod
spec:
  containers:
    - name: busybox
      image: busybox
      command: ['sh', '-c', 'echo "Success!" > /output/success.txt']
      volumeMounts:
      - name: pv-vol
        mountPath: "/output"
  volumes:
    - name: pv-vol
      persistentVolumeClaim:
        claimName: my-pvc
```




## 5. Troubleshooting 30%  

### 5.1. Evaluate cluster and node logging  
https://kubernetes.io/docs/tasks/debug-application-cluster/troubleshooting/  
https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/  

```
kubectl get nodes
kubectl describe node <name>
kubectl cluster-info dump
```

Deployed with Kubeadm
```
kubectl get pods -n kube-system
kubectl describe pod <name> -n kube-system
```

Deployed without Kubeadm
```
systemctl status kubelet | docker | ...
systemctl start kubelet
systemctl enable kubelet
```

#### Cluster and Node Logs
https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/#looking-at-logs
```
sudo journalctl -u kubelet   ### -u = unit
sudo journalctl -u docker | containerd
```

Deployed with Kubeadm
```
kubectl logs <pod> -n kube-system
```
Deployed without Kubeadm
```
/var/log/kube-apiserver.log
/var/log/kube-scheduler.log
/var/log/kube-controller-manager.log
```

### 5.2. Understand how to monitor applications   
https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/
https://kubernetes.io/docs/tasks/debug-application-cluster/debug-running-pod/

```
kubectl get pods
kubectl describe pod <name>
```

CLI commands within a Pod
```
kubectl exec <pod> -c <container> -- <command>
```

verify there are endpoints for a service
```
kubectl get endpoints <service>
```


### 5.3. Manage container stdout & stderr logs  

stdout + stderr streams
```
kubectl logs <pod> -c <container> 
```

### 5.4. Troubleshoot application failure  

### 5.5. Troubleshoot cluster component failure   

### 5.6 Troubleshoot networking  

https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/
https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/

Netshoot
https://github.com/nicolaka/netshoot




