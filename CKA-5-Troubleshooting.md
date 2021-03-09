 
# 5. Troubleshooting 30%  


Domain	| Weight
------- | -------------
**5. Troubleshooting**	| 30%  
[5.1. Evaluate cluster and node logging](CKA-5-Troubleshooting.md#51-evaluate-cluster-and-node-logging)  |  
[5.2. Understand how to monitor applications](CKA-5-Troubleshooting.md#52-understand-how-to-monitor-applications)  |  
[5.3. Manage container stdout & stderr logs](CKA-5-Troubleshooting.md#53-manage-container-stdout--stderr-logs)  |  
[5.4. Troubleshoot application failure](CKA-5-Troubleshooting.md#54-troubleshoot-application-failure)  |  
[5.5. Troubleshoot cluster component failure](CKA-5-Troubleshooting.md#55-troubleshoot-cluster-component-failure)  |  
[5.6 Troubleshoot networking](CKA-5-Troubleshooting.md#56-troubleshoot-networking)  |  

<br/>

## 5.1. Evaluate cluster and node logging  
https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/#looking-at-logs  

```
sudo journalctl -u kubelet   ### -u = unit
sudo journalctl -u docker | containerd
```

To list all units available for journalctl to use:
```
systemctl list-unit-files --all
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


## 5.2. Understand how to monitor applications   
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

Verify there are endpoints for a service
```
kubectl get endpoints <service>
```


## 5.3. Manage container stdout & stderr logs  
https://kubernetes.io/docs/concepts/cluster-administration/logging/

stdout + stderr streams
```
kubectl logs <pod> -c <container> 
```

## 5.4. Troubleshoot application failure  
https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/

Check  current state of the Pod and recent events:
```
kubectl describe pods <podname>
```

Verify there are endpoints for a service
```
kubectl get endpoints <service>
```

Check k8s scheduler:
```
kubectl get events
```

## 5.5. Troubleshoot cluster component failure   
https://kubernetes.io/docs/tasks/debug-application-cluster/troubleshooting/  
https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/  

```
kubectl get nodes
kubectl describe node <name>
kubectl cluster-info dump
```

For cluster deployed with Kubeadm:
```
kubectl get pods -n kube-system
kubectl describe pod <name> -n kube-system
```

For cluster deployed without Kubeadm:
```
systemctl status kubelet | docker | ...
systemctl start kubelet
systemctl enable kubelet
```

## 5.6 Troubleshoot networking  
https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/

## Debugging Calico
```
kubectl get deploy calico-kube-controllers -n kube-system -o wide
kubectl get po -n kube-system -o wide | grep calico
kubectl logs <calico-kube-controllers-podname> -n kube-system
kubectl logs <calico-node-podname> -n kube-system
```

### Debugging DNS 
https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/

Check CoreDNS:
```
kubectl get deploy coredns -n kube-system -o wide
kubectl get po -n kube-system -o wide | grep coredns
kubectl logs <coredns-podname> -n kube-system
```

```
kubectl apply -f https://k8s.io/examples/admin/dns/dnsutils.yaml

kubectl get pods dnsutils

kubectl exec -it dnsutils -- nslookup kubernetes.default
kubectl exec -it dnsutils -- cat /etc/resolv.conf
```

### Debugging kube-proxy

Check if kube-proxy is running:
```
ps auxw | grep kube-proxy
```

Check kube-proxy logs:
```
/var/log/kube-proxy.log
```


### Cool tools

Netshoot  
https://github.com/nicolaka/netshoot




