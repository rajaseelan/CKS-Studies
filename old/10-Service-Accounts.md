# Service Accounts - Cluster Hardening

*ServiceAccount* is managed by k8s. It is a resource.  
*User* is not a k8s resource. Management is passed to third party providers (Authentication).

* Namespaced
* **default** ServiceAccount in every namespace
* ServiceAccount has a *secret*. Can be used to talk to k8s API.

## Lab

### Run Pod using custom ServiceAccount
1. Create a service account accessor
2. Inspect toekn for accessor
3. create a pod with a servicecaccount
4. Test query API with newly created service account

```
curl https://kubernetes -k -H "Authorization: Bearer $(cat /run/secrets/kubernetes.io/serviceaccount/token)" 
```

Service accounts are name in this format: `system:serviceaccount:default:accessor`
### Disable ServiceAccount mounting
* Pod doesn't need to talk to the k8s api most of the time
* Remove mounting of ServiceAccount Token / secrets
* Accomplished 2 ways  
  1. Set in ServiceAccount  
      apiVersion: v1
      kind: ServiceAccount
      metadata:
        name: build-robot
      automountServiceAccountToken: False

  1. Pod Spec
  ```
      apiVersion: v1
      kind: Pod
      metadata:
        name: my-pod
      spec:
        serviceAccountName: accessor
        automountServiceAccountToken: false
  ```
### Limit ServiceAccount with RBAC
1. Test service account - able to delete secrets?
```
kubectl auth can-i delete secret --as system:serviceaccount:default:accessor
```

2. Create a clusterrolebinding to the edit ClusterRole
```
kubectl create clusterrolebinding accessor --clusterrole=edit --serviceaccount default:accessor
```

