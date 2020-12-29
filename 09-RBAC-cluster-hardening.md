# RBAC - Role Based Access Control

## Enable RBAC
Startup the `kube-apiserver` with the option included `--authorization-mode`. In a kubeadm installed cluster, the static pod manifest has the options: `--authorization-mode=Node,RBAC` enabled.

Combines `Roles` and `Bindings`.  
Default Deny policy.  
Explicitly specify what is allowed.  

Has `namespaced` and `non-namespaced` resources.  

Get a list of namespaced resources:  
```
kubectl api-resources --namespaced=true
```

Get a list of non-namespaced resources.
```
kubectl api-resources --namespaced=false
```

Available Permissions
* Role
  * In a single Namespace

* ClusterRole
  * ClusterWide
  * non namespaced objects

How to Apply Permissions
* RoleBinding
  * single namespace

* ClusterRoleBinding
  * all namespaces
  * non-namespaced objects

## Possible combinations of Permissions
Type | Combination
--- | ---
namespaced | Role + Rolebinding + User / Group / ServiceAccount
non-namespaced | ClusterRole + ClusterRoleBinding + User / Group / ServiceAccount
namespaced (Selective) | ClusterRole + RoleBinding + User / Group / Service Account

**NOTE** You can create a rolebinding from a Cluster Role, that will *limit* a ClusterRole to a Namespace

## Test RBAC Rules

### Roles
* Create 2 namespaces, `red` and `blue`
* User `jane` can `get` `secrets` in namespace `red`
* User `jane` can only `list`, `get` secrets in namespace `blue`
* Test using `auth can-i`


### ClusterRoles
* Create clusterrole `deploy-deleter` to `delete` `deployments`
* User `jane` can delete deployments in `all namespaces`
* User `jim` can delete deployments in namespace `red`

# Accounts in k8s

## ServiceAccounts
* ServiceAccount resource
* managed by k8s api
* User has
  * private key
  * certificate - signed by k8s CA
  * username denoted by `common name`, specified as `/CN=jane` during cert / key creation

* Workflow
  * openssl (cert) -> CSR (CertificateSingingRequest - k8s API) -> -> Approve (k8s API) -> Generates Certificate (CRT)

* No way to invalidate cert

When Cert is compromised:  
* Remove all access via RBAC
* Username cannot be used until cert expired
* Create a new CA and resign all CA

## User
* no k8s resource
* a third party service manages users

### LAB
* create CSR
* Sign CSR using k8s API
* Use cert + key to connect to k8s API

Solution
1. generate key
```
openssl genrsa -out jane.key 2048 
```

2. Create CSR
```
openssl req -new -key jane.key -out jane.csr 

# fill in the Common Name  - CN
```

3. Create k8s CertificateSigningRequest
```
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: jane
spec:
  groups:
  - system:authenticated
  request: <csr in base64 encoding>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
```
base64 -w 0 jane.csr - to get a single line

4. Approve CertificateSigningRequest
```
kubectl certificate approve jane
```

5. Extract signed certificate
```
kubectl get csr jane -o yaml

# extract the status/certififcate field
```

6. Use Certificate
```
# Create user jane in kubeconfig

users:
- name: jane
    client-certificate-data: XXXXX
    client-key-data: XXXXXX

# create context 
contexts:
- context:
    cluster: kubernetes-prod
    user: jane
    name: jane@kubernetes-prod

kubectl config set-credentials jane --client-key=jane.key --client-certificate=jane.crt

# put cert data in
kubectl config set-credentials jane --client-key=jane.key --client-certificate=jane.crt --embed-certs
```

* Connect user jane with cluster kubernetes
```
kubectl config set-context jane --user=jane --cluster=kubernetes
```

* switch to user jane
k config use-context jane
