# Architecture

container in container engine.

## Container Engine responsibilities:
* Run container
* Handle Volumes
* Allow common deployment techniques
* Allow container communication
* Keep containers alive, health checks
* Schedule containers efficiently

## Communication
```
kubectl -> apiserver -> worker (kubelet) -> pod
             ^                 (kube-proxy) - create services
             |
            etcd
            scheduler
            Controller Manager
            Cloud COntroller Manager

```

## Pod-to-Pod communication
* managed by the CNI
* every pod can communicate w/ every pod
* no NAT required
* every pod can communicate w/ every pod across nodes

## PKI - Public Key Infrastructure
* CA - Certificate authority - the trusted root of all CAs
  * All cluster certs signed by CA
  * Used by components to validate each other

## Finding Certificates for k8s components
Cert | Comment
--- | ---
Cert locations | `/etc/kubernetes/pki`
etcd certs | `/etc/kubernetes/pki/etcd`
Certificate Authority | `/etc/kubernetes/pki/ca/crt`
API Server Cert | `/etc/kubernetes/pki/apiserver.crt`
API Server etcd  client | `/etc/kubernetes/pki/apiserver-etcd-client.crt`
API Server kubelet client | `/etc/kubernetes/pki/apiserver-kubelet-client.crt`
etcd server | `/etc/kubernetes/pki/etcd/server.crt`
scheduler | `/etc/kubernetes/scheduler.conf`
controller-manager | `/etc/kubernetes/controller-manager.conf`
kubelet | `/var/lib/kubelet/pki/kubelet-client-current.pem`
kubelet server certificate | `/var/lib/kubelet/pki/kubelet.crt`
