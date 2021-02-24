
# 3. Services & Networking 20%

Domain	| Weight
------- | -------------
**3. Services & Networking** 	| 20%  
[3.1. Understand host networking configuration on the cluster nodes](CKA-3-Services-Networking.md#31-understand-host-networking-configuration-on-the-cluster-nodes)  |  
[3.2. Understand connectivity between Pods](CKA-3-Services-Networking.md#32-understand-connectivity-between-pods) |  
[3.3. Understand ClusterIP, NodePort, LoadBalancer service types and endpoints](CKA-3-Services-Networking.md#33-understand-clusterip-nodeport-loadbalancer-service-types-and-endpoints) |  
[3.4. Know how to use Ingress controllers and Ingress resources](CKA-3-Services-Networking.md#34-know-how-to-use-ingress-controllers-and-ingress-resources) |  
[3.5. Know how to configure and use CoreDNS](CKA-3-Services-Networking.md#35-know-how-to-configure-and-use-coredns) |  
[3.6. Choose an appropriate container network interface plugin](CKA-3-Services-Networking.md#36-choose-an-appropriate-container-network-interface-plugin) | 


## 3.1. Understand host networking configuration on the cluster nodes  
https://kubernetes.io/docs/concepts/cluster-administration/networking/

Each Pod has its own unique IP address within the cluster 

By default, a single virtual network exists allowing all pods to communicate with each other.

<br/><br/>

## 3.2. Understand connectivity between Pods  
By default, pods are non-isolated; they accept traffic from any source.

### NetworkPolicy  
https://kubernetes.io/docs/concepts/services-networking/network-policies/  
If any networkPolicy selects a Pod, then the Pod is completely isolated and will only be open to traffic allowed by networkPolicy.

* podSelector = which pod to apply the networkPolicy
* ingress = into the Pod
* egress = leaving the Pod
<br/>

#### Default deny all ingress traffic 
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
<br/>

#### Default allow all ingress traffic 
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
<br/>

#### Sample Network Policy
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

<br/><br/>

## 3.3. Understand ClusterIP, NodePort, LoadBalancer service types and endpoints  
https://kubernetes.io/docs/concepts/services-networking/service/

Service = expose apps running as a set of Pods 
Service routing = load balancing from Client to Pods
Endpoints = Pods servicing the apps

Service Types:
* ClusterIP = expose apps inside cluster
* NodePort = expose apps outside cluster
* LoadBalancer = expose apps outside using external load balancer (eg. AWS ALB…)
* ExternalName (CKA out of scope)
<br/>

### ClusterIP
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
<br/>

### NodePort
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
<br/>

### DNS for Services and Pods   
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

<br/><br/>

## 3.4. Know how to use Ingress controllers and Ingress resources  
https://kubernetes.io/docs/concepts/services-networking/ingress/
https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/

Ingress objects manage external access to the services in a cluster, typically HTTP. 

* Ingress may provide load balancing, SSL termination and name-based virtual hosting.
* Requires an Ingress Controller
* Routing Rules with a set of Paths, each with a backend to a Service
* If Service uses a Named Port, then Ingress can refer to the Named Port (‘web’) instead of number (’80’).


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

<br/><br/>

## 3.5. Know how to configure and use CoreDNS  
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
```json
<service-name>.<namespace>.svc.<cluster-domain>
svc-frontend.default.svc.cluster.local
```

<br/><br/>  
  
## 3.6. Choose an appropriate container network interface plugin  
https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/

K8s nodes will remain “NotReady” until a network plugin is installed. 

Each CNI plugin has its own implementation and own installation procedure. 

### Calico
https://docs.projectcalico.org/

Installing Calico:
```
kubectl apply -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml
```
<br/>

### Flannel
https://github.com/flannel-io/flannel

Installing Flannel:
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
