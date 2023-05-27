# Restrict API Access

* Authentication
* Authorization
* Admission

## Lifecycle of a Request
i.e Create a Pod.  

1. Authentication
  Who are you?
    User / ServiceAccount.  

1. Authorization:  
  Are you applowed to perform action?

1. Admission Control
  Has Pod Limit been reached?
  Other type of controllers

## Security Measures
1. Don't allow anonymous access
2. Close insecure Port
3. Don't expose ApiServer to the outside
4. Restrict Access from Nodes to API (NodeRestriction)
5. Prevent Unauthorized Access (RBAC)
6. Prevent pods from accessing API
7. APIServer port behind firewall / allowed cloud provider IP ranges

## Practice

### Restrict API Access  
kube-apiserver must be started with `kube-apiserver --anonymous-auth=false`
  * RBAC / ABAC requires explicit authorization for anonymous user

1. Check kube-apiserver static pod manifest
1. Check if anonymous access is disabled  
    `curl -k https://localhost:6443`
    If you get a response with `"message": "forbidden: User \"system:anonymous\" cannot get path \"/\""` -anonymous access enabled, but blocked.
1. Edit kube-apiserver manifest. after pod is relaunched, an anonymous curl will give this error:
```
curl -k https://localhost:6443
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {
    
  },
  "status": "Failure",
  "message": "Unauthorized",
  "reason": "Unauthorized",
  "code": 401
}
```

### Insecure Access
* kubectl sends a *HTTPS* request to kube-apiserver
* configure `kube-apiserver --insecure-port=8080` to allow *HTTP* access to API.
  * **ALERT** Insecure access bypasses authentication / authorization modules

#### Lab - enable insecure access
* edit kube-apiserver.yaml
* curl http://localhost:8080


### Lab Manual API Request
Perform a manual API Request using info from kubeconfig file.  

* Display kubectl config file  
<details>
kubectl config view
</details>

* Display kubeconfig with credentials  
<details>
`kubectl config view --raw`
</details>

* Create a request manually

<details>

<summary>How to assemble a crul request manually</summary>

1. Extract CA Cert data from kubeconfig

```
# you should get the client private key from this
echo "certificate-authority-data gobbledegook" | base64 -d > ca
```

2. Extract the signed client certificate data
```
# You will get the signed client cert file
echo 'client-certificate-data gobbledeygook' | base64 -d > crt
```

3. Extract client private key
```
# You will get the client private key
echo 'client-key-data gobbledey-gook' | base64 -d > key
```

4. Get the server address (kube-apiserver)
```
# it is the server: section of the kubeconfig
# server: https://192.168.123.19:6443
```
```
5. Assemble a curl command with the data
curl https://192.168.123.19:6443 --cacert ca --cert crt --key key
```
</details>

### External API server Access
Make the Kubernetes API Publicly accessible.  

<details>
  <summary>Edit the kubernetes API service to become a NodePort</summary>

    # change from ClusterIP to NodePort  
    kubectl edit svc kubernetes

</details>

**Note** a kubeconfig command to a IP that is not part on the cert's CN will fail.

<details>

<summary>How to inspect certificate using ssl</summary>

    # Note the CN (Common Names), and 
    #
    # x509 Subject Alternative Name DNS:
    openssl x509 -in apiserver.crt -text

    # add an entry to the hosts file as DNS
    # or curl with the --resolve parameter

</details>

## Node Restriction

* Admission Controller
<details>

<summary>How to enable NodeRestriction in the kubernetes</summary>

    # part of kube-apiserver startup parameters
    kube-apiserver --enable-admission-plugins=NodeRestriction

</details>

<details> 
    <summary>What does a NodeRestriction Controller do?</summary>

* Limits the Node Labels a kubelet can modify
* Only Modiy own node's labels
* Only modify Labels running on pods in its node

This ensures secure workload Isolation via labels
No one can pretend to be a secure workload / secure pods

</details>  

### Lab: Check Worker Node Restriction

* Where is the kubelet kubeadm config file?
  <details>

  ```
  /etc/kubernetes/kubelet.conf
  ```

  </details>

* use kubelet kubeconfig to perform api requests
  <details>

    ```
    export KUBECONFIG=/etc/kubernetes/kubelet.conf
    k get nodes # works 

    k label node cks-master some=label # doesn't work
    k label node node-name some=label # works work
    ```

  </details>

* What labels are restricted from kubelet?
  <details>

    ```
    # any labels that start with node-restriction.kubernets,io
    node-restriction.kubernetes.io/test=yes
    ```

  <details>
  