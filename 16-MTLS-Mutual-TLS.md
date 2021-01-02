# Mutual TLS (mTLS)

* What is mTLS (pod-to-pod communication)

* Service Meshes

* Scenarios

## mTLS
* two-way authentication
* Both parties authenticate at the same time.

### Scenario for mTLS

* Ingress Traffic secured by https is usually terminated at the ingress
* Pod to Pod communication is guaranteed by the CNI
* An attacker who spins up a pod can communicate with any pod without boundaries, perform a MiTM and sniff traffic
* mTLS encrypts traffic from / to pods


<details>
<summary>How to encrypt Pod-to-Pod traffic</summary>

Pods need client / server certificate _each_.  
A trusted CA to sign the certs.  
Certs must be rotated

</details>

<details>
<summary>How to reduce complexity / overhead in maintaining mTLS certs?</summary>

Add a Proxy (sidecar) container  
Proxy container can be managed externally.  
The manager can be a Service Mesh like Istio / Linkerd  

</details>

<details>
<summary>How does the Sidecar Proxy get traffic from app pod?</summary>

* an initContainer creates iptables rules (NET_ADMIN capability)
* traffic redicted to proxy sidecar
</details>

### Lab: Create a Sidecar Proxy

<details>
<summary>Create a Sidecar Proxy with NET_ADMIN capability</summary>

```
Create an initContainer that installs iptables.
Makesure it has NET_ADMIN capability

```
</details>


Articles to read:  
* Demystifiying Istio's sidecar model