# Cluster Hardening - Upgrade K8s

## k8s Release Cycles

### Understanding k8s Versioning
1.19.2  
1 - Major Version  
19 - Minor Version
2 - Patch Version

Maintenance releases for the most recent three minor releases. i.e (1.20, 1.19, 1.18).  


## Cluster Upgrade Workflow
1. Upgrade Master COmponenets
  - apiserver
  - controller-manager
  - scheduler
2. Worker Componenets
  - kubelet
  - kube-proxy  
  **NOTE** Worker componenets can be 2 minor versions behind. *Not recommended)*

Components should be the same minor version as apiserver.

## Upgrade Commands
<details>
  <summary>1. Evict pods from nodes</summary>
  
    # standard command
    kubectl drain
  
</details>

<details>
  <summary>2. Mark node unschedulable</summary>

    kubectl cordon <nodename>


</details>
3. Upgrade System
<details>
  <summary>Uncordon Node</summary>

    kubectl uncordon node

</details>

### Application Considerations

Application should be made aware of the following to work best in a k8s world:  
<details>
  <summary>Applications should adhere to k8s features</summary>

    1. Pod gracePeriod / Terminating state
    2. Pod Lifecycle Events
    3. PodDisruptionBudget

</details>

### Lab: Upgrade Kubernetes
<details>
  <summary>Upgrade Master</summary>

    1. kubeadm drain cks-master --ignore-daemonsets
    2. apt-cache show kubeadm | grep 1.19 # Get latest version of k8s
    3. apt install kubeadm=1.19.6-00 kubelet=1.19.6-00 kubectl=1.19.6-00
    4. sudo kubeadm upgrade plan
    5. kubeadm upgrade apply v1.19.6
    6. kubectl get node # verify master is ok
    7. kubectl uncordon cks-master

</details>

### Lab: Upgrade Workers
<details>
  <summary>Upgrade Worker</summary>

    1. kubectl drain cks-worker
    2. apt install kubeadm=1.19.6-00
    3. kubeadm version
    4. kubeadm upgrade node 
    5. apt install kubelet=1.19.6-00 kubectl=1.19.6-00 
    6. kubectl uncordon cks-worker

</details>