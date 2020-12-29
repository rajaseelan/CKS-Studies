# Network Policies

* Firewall for k8s
* CNI plugins that support Network Policies
  * Calico
  * Weave
* Namespace level
* Ingress / Egress for selectors

## Pod communications
* unrestricted by default (k8s default)

## Creating a Network Policy
1. if a pod is selected, **only that traffic** will be allowed
1. selectors available
    1. NameSpace Selector
    1. PodSelector
    1. Ip Blocks


## Deny all Network Policy

* Deny all outgoing because pod is selected, but no trafffic specs.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: example
  namespace default
spec:
  podSelector:
    matchLabels:
      id: frontend
  policyTypes:
  - Egress
```


```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: example
  namespace default
spec:
  podSelector:
    matchLabels:
      id: frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          id: ns1
    ports:
    - protocol: TCP
      port: 80
  - to:
    - podSelector:
        matchLabels:
          id: backemd
          
```