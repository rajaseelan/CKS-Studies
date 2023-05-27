# OS Level Security Domains
* Security Contexts
* Pod Security Polices
* Scenarios

## Security Context
* privilledge / access control for Pod / Container
* userid / groupid
* privileged / unpriviledged (Linux Capabilities)

Pod level SecurityContext.  
All containers run as the User/ Group
```yaml
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 2000
  containers:
    - name: nginx
      image: nginx
```
* override on a per container basis.

Parameters  | Description
--- |---
`runAsUser` | Primary user
`runAsGroup`| Group
`fsGroup`   | Additional groups

 Read up on `SecurityContexts` in Documentation.  
 Lookup up the task **Configure a Security Context for a Pod or Container**

 ### Lab: Set Container User and Group
 <details>
<summary>Create a Pod running as User / Group / Fsgroup 1000 /1000 / 3000</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 3000
  containers:
  - name: nginx
    name: nginx
```

 </details>

 ### Lab: Container Not Allowed to run as root
  <details>
<summary>Configure the Container to run as a Non Root User</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 3000
  containers:
  - name: nginx
    name: nginx
    securityContext:
      runAsNonRoot: true
```

 </details>

 <details>
 <summary>What happens if runAsNonRoot is configured but there are no explicit user / group IDs set?</summary>

 Pods created will not run with a `CreateContainerConfigError`  
 A kubectl describe will havethe following in logs:  
 ```
 Error: container has runAsNonRoot and image will run as root`
 ```

 </details>

---

 ### Lab: Priviledged Containers
* run unprivileged by default
* Run as privileged to
  * Access Devices
  * Run Docker deamon inside container
* <details><summary>How to Run as privileged Container</summary>docker run --privileged</details>
<details><summary>What does Privileged mean></summary>  Container user 0 is mapped to host user 0 (root)</details>

<details>
<summary>Configure a container in pod to run as priviledged</summary>

```yaml
spec:
  containers:
  - name: nginx
    image: nginx
    securityContext:
      privileged: true
```
</details>

<details>
<summary>Test Privileged Pod by setting File System</summary>

```
kubectl exec -it priv-pod -- /bin/sh
sysctl kernel.hostname=new-host
```

</details>

---

## Privilege Escalation

<details>
<summary>What enables a process in a pod to gain more privileges than its parent process?</summary>

```
# Set in the Pod / Container Spec
# enabled by default!!!
allowPrivilegeEscalation: true
```

</details>

<details>
<summary>When is this turned on by default?</summary>

```
# If Pod / Container Spec has 
Privileged: true

# or 
capabilities: [ "CAP_SYS_ADMIN" ]
```

</details>

### Lab: Disable Privilege Escalation

<details>
<summary>Disable Priviledge Escalation in Container</summary>
```yaml
spec:
  containers:
  - name: nginx
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
```
</details>

---

## PodSecurityPolices

* CLuster level resource
* Control which security conditions a Pod has to run
* Admission controller, must be enabled in the admission plugins

#### PSP Creation Workflow
1. Admin create PodSecuritypolicy
2. Pods **must** meet polocies before being created

<details>
<summary>Enable PodSecurityPolices in Cluster</summary>

```
containers:
  - command:
    - kube-apiserver
    - --enable-admission-plugins=PodSecurityPolicy
```

</details>

<details>
<summary>What are possible PodSecurityPolicy Options</summary>

- privileged
- hostPID, hostIPC
- hostNetwork, hostPorts
- volumes
- allowedHostPaths
- fsGroup
- readOnlyRootFilesystem

</details>

### Lab: Enable PodSecurityPolicy

<details>
<summary>Enable PodSecurityPolicies in the Cluster</summary>

```
# in /etc/kubernetes/manifests/
containers:
  - command:
    - kube-apiserver
    - --enable-admission-plugins=PodSecurityPolicy
```

</details>

<details>
<summary>Create a PodSecurityPolicy disabling priviledge escalation</summary>

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: default-psp
spec:
  allowPrivilegeEscalation: false
  privileged: false  # Don't allow privileged pods!
  # The rest fills in some required fields.
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  runAsUser:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  volumes:
  - '*'

```
</details>

**NOTE**: Once you create a PodSecurityPolicy, only Deployment that _can use_ a PSP are allowed to run (if they comply with a PSP)

<details>
<summary>How to enable serviceaccounts to use PodSecurityPolicy?</summary>

```
k create role psp-access --verbs=use --resource=podsecuritypolicies
k create rolebinding psp-access --role=psp-access --serviceaccount=default:sa
```

</details>

**NOTE**: `PodSecurityPolicy` Documentation is a good read.