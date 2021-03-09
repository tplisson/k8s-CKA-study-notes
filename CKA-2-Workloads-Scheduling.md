# 2. Workloads & Scheduling 15%

Domain	| Weight
------- | -------------
**2. Workloads & Scheduling**	| 15%  
[2.1. Understand deployments and how to perform rolling update and rollbacks](CKA-2-Workloads-Scheduling.md#21-understand-deployments-and-how-to-perform-rolling-update-and-rollbacks) |  
[2.2. Use ConfigMaps and Secrets to configure applications](CKA-2-Workloads-Scheduling.md#22-use-configmaps-and-secrets-to-configure-applications) |  
[2.3. Know how to scale applications](CKA-2-Workloads-Scheduling.md#23-know-how-to-scale-applications) |  
[2.4. Understand the primitives used to create robust, self-healing, application deployments](CKA-2-Workloads-Scheduling.md#24-understand-the-primitives-used-to-create-robust-self-healing-application-deployments) |  
[2.5. Understand how resource limits can affect Pod scheduling](CKA-2-Workloads-Scheduling.md#25-understand-how-resource-limits-can-affect-pod-scheduling) |  
[2.6. Awareness of manifest management and common templating tools](CKA-2-Workloads-Scheduling.md#26-awareness-of-manifest-management-and-common-templating-tools) |  

</br></br>


## 2.1. Understand deployments and how to perform rolling update and rollbacks  
https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/

### Deployments  
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
* Scaling up / down easily number of replicas
* Rolling updates to deploy new SW version  

<br/>

Here's an example of a deployment for an NGINX app:
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
<br/>

### Rolling Updates  
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment

Three options for rolling updates:

1- Using imperative command `kubectl set image`:
(use `--record` to easily rollback)
```
kubectl set image deployment/my-deployment nginx=nginx:1.19.2 --record
```

2- Using imperative command `kubectl edit deploy`:
```
kubectl edit deployment my-deployment
```
Note: any change is applied immediately, no need for `kubectl apply`

3- Editing specs in the deployment YAML manifest:
```
vi deployment.yaml
kubectl apply -f deployment.yaml
```

Check deployment status:
```
kubectl rollout status deployment my-deployment
kubectl rollout status deployment/my-deployment
kubectl rollout history deployment my-deployment
```

### Rollback  
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment

Three options for rolling back:

1- Using imperative command `kubectl rollout undo`:
```
kubectl rollout history deployment/my-deployment
kubectl rollout undo deployment.v1.apps/my-deployment
kubectl rollout undo deployment.v1.apps/my-deployment --to-revision=1
```
Note: use `--to-revision` when `—-record` was used


2- Using imperative command `kubectl edit deploy`
```
kubectl edit deployment my-deployment
kubectl rollout status deployment my-deployment
```
Note: any change is applied immediately, no need for `kubectl apply`


3- Editing specs in the deployment YAML manifest:
```
vi deployment.yaml
kubectl apply -f deployment.yaml
```

<br/>

Check rollout status:
```
kubectl rollout status deployment my-deployment
kubectl rollout history deployment my-deployment
```

<br/><br/>


## 2.2. Use ConfigMaps and Secrets to configure applications

### ConfigMaps
https://kubernetes.io/docs/concepts/configuration/configmap/  

ConfigMap is an API object used to store non-confidential data in key-value pairs. 

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

Secrets are API objects used to store and manage sensitive information, such as passwords, OAuth tokens, and SSH keys.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: opaque       # opaque = arbitrary user-defined data
data:
  secretkey1: <base64-string1>
```

Or using the imperative command:
```
kubectl create secret generic my-secret --from-file=path/to/bar
```

How to get a base64 key from a string:
```
echo -n 'secret' | base64 
```  

**Note**: the `-n` flag for `echo` means: "do not output the trailing newline".

<br/>

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
<br/>

### Configuration Volumes  
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

<br/><br/>

## 2.3. Know how to scale applications  
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment

Three options for scaling an app:

1- Using imperative command `kubectl scale`:
```
kubectl scale deployment.v1.apps/my-deployment --replicas=5
```
<br/>

2- Using imperative command `kubectl edit deploy`:
```
kubectl edit deployment my-deployment
```
Note: any change is applied immediately, no need for `kubectl apply`
<br/>

3- Editing replicas number in the deployment YAML manifest: 
```
kubectl apply -f deployment.yaml
```
<br/><br/>


## 2.4. Understand the primitives used to create robust, self-healing, application deployments   
https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/
https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

### Liveness Probes  
Testing whether the container is running.

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
<br/>

### Startup Probes  
Testing whether the application within the container is started.
* Useful for Pods that have containers that take a long time to come into service.
* allowing a time longer than the liveness interval would allow.

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
<br/>

### Readiness Probes   
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
<br/>

### Restart policy  
https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy

#### Always  
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

#### OnFailure  
Restart ONLY IF the container exits w an error code or determined unhealthy with liveness probe.

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

#### Never  
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
<br/><br/>


### Multi-Container Pods  
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
<br/>

#### Init-Container  
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
<br/><br/>


## 2.5. Understand how resource limits can affect Pod scheduling   
https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-requests-and-limits-of-pod-and-container  
  
  
### Resource Limits  

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
<br/>
  
### K8s Scheduling  
https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/  
https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/  

Using `nodeName`: only run the Pod on this specific node:
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

Using `nodeSelector`: run the Pod on any node with matching label(s):
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

Obviously the label must be included in the node definition:
```yaml
apiVersion: v1
kind: Node
metadata:
  labels:
    disk: fast
```
<br/>

### DaemonSets  
https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/  
A DaemonSet ensures that all (or some) nodes run a copy of a Pod.

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
<br/>

### Static Pods   
https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/  
Pod managed directly by Kubelet, not by K8s API server

Mirror Pod created to represent a Static Pod in the Kuberntes API, allowing you to easily view the Static Pod's status. But no change can be done to them via API. 

Manifest path for static pods on each node under:
```
vi /etc/kubernetes/manifests/<podname>
```

Restart kubelet on the node where the kubelet is running:
```
systemctl restart kubelet
```

Check result on the Control Plane:
```
kubectl get pods
```
<br/>

## 2.6. Awareness of manifest management and common templating tools   
https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/

* [helm](https://helm.sh)  
  * Templating (charts) & package mgmt  
* [kcompose](https://github.com/kubernetes/kompose)  
  * from Docker compose to K8s objects  
* [kustomize](https://kustomize.io)  
  * Config managment tool (similar to helm)  
  * https://kubernetes.io/blog/2018/05/29/introducing-kustomize-template-free-configuration-customization-for-kubernetes/ 

