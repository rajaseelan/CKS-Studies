# Container Runtime Sanboxes
* Just because it runs in a container, doesn't mean it is more protected.
* Processes share the same kernel
* Containers are just processes in a different namespace

## Sandboxes
* Test Environment
* Isolated from the rest of the system

## Containers
* Communicate w/ OS Kernel via **system calls**

### User Space
* User Processes
* Uses System Calls

### Kernel Space
* Communicates with hardware
* Uses system calls to communicate w/ Processes

### Prevent User Space from directly accessing system calls
* Introduce a **SandBox** Layer
* Intercepts / filters calls reaching Kernel
* Tradeoff: Overhead in resources
* Application dependent (syscall heavy apps might not be appropriate.)

## Labs

<details>
<summary>Call Linux Kernel from within Container</summary>

```
1. Create a pod 
kubectl run nginx --image=nginx

2. Exec into pod
kubectl exec -it nginx -- /bin/bash

3. exec a uname -r 

4. Exit pod & run uname -r 
uname -r # notice similar versions

5. observe syscalls made by doing an strace
strace uname -r 

uname({sysname="Linux", nodename="cks-master", ...}) = 0
fstat(1, {st_mode=S_IFCHR|0600, st_rdev=makedev(136, 0), ...}) = 0
write(1, "5.4.0-1029-gcp\n", 155.4.0-1029-gcp
)

Example exploit: search for 'Dirty Cow'
```

</details>

---
## OCI - Open Container Initiative

* By Linux Foundation
* Open Standards for Virtualization
* Specifications for
  * runtime
    * Linux Foundation reference runtime: **runc**
  * image
  * distribution

* `kubelet` can only run with a single Container Runtime Interface at a time.
* How do you sepcify which runtime to use?
  <details>
  
  ```
  kubelet --container-runtime {string}
          -container-runtime-endpoint {string}
  ```

  </details>

### Labs
<details>
<summary>Explore crictl</summary>

```
crictl ps # show the static pods launched by the kubelet

# pull image
crictl pull

# crictl is k8s native - can list k8s pods
crictl pods
```

</details>

---
## Container Runtime Sandboxes: katacontainers
* additional lightweight kernels on top of linux kernel
* lightweight VM
* strong separation layer
* every container in own private VM (hypervisor based)
* QEMU - default hypervisor
  * nested virtualization required (for cloud)

---

## Container Runtime Sandboxes: gVisor

* user space kernel for containers
* simulates kernel syscalls w/tih limited functionality
* runtime named `runsc`

### Lab: Runtime Class
* A k8s resource that specifies a different runtime handler
* Create runtime class
* Specify in pod spec that it should use the runtime class

<details>
<summary>Configure a new Runtime Class for k8s</summary>

1. create a Runtime Class

```yaml
apiVersion: node.k8s.io/v1  # RuntimeClass is defined in the node.k8s.io API group
kind: RuntimeClass
metadata:
  name: gvisor  # The name the RuntimeClass will be referenced by
  # RuntimeClass is a non-namespaced resource
handler: runsc  # The name of the corresponding CRI configuration
```

2. Create a pod with a *generic* runtime class
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: gvisor
  name: gvisor
spec:
  runtimeClassName: myclass
  containers:
  - image: nginx
    name: gvisor
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

3. Run it - it won't work
```
Error from server (Forbidden): error when creating "gvisor.yaml": pods "gvisor" is forbidden: pod rejected: RuntimeClass "myclass" not found
```

4. Modify RuntimeClass to `gvisor` and run the pod
It *should* work, but you need a bunch of nodes that have gvisor installed.


a `kubectl describe` shows the problem:  
```
Events:
  Type     Reason                  Age               From               Message
  ----     ------                  ----              ----               -------
  Warning  FailedCreatePodSandBox  2s (x2 over 17s)  kubelet            Failed to create pod sandbox: rpc error: code = Unknown desc = RuntimeHandler "runsc" not supported
```
</details>

### Lab: Install gVisor runtime
```
        kubelet

        containerD

    runc            runsc(gvisor)
```

Upon configuring, the `kubelet` args woul be like:  
```
KUBELET_EXTRA_ARGS="--container-runtime remote --container-runtime-endpoint unix:///run/containerd/containerd.sock"
```

## Relevant Talks

* OCI, CRI, ??? Making Sense of the Container Runtime Landscape
* Sanboxing your containers with gVisor
* Kata Containers An Introduction and Overview
