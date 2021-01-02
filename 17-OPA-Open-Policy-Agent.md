# Open Policy Agent

* customized policies
* Intro to OPA/ Gatekeeper
* Enforce Labels
* Enforce Pod Replica Count

**Typical Workflow**  
```
# kubectl crreate pod

1. Authentication
   Who are you?
     user / serviceAccount

2. Authorization
Allowed to create pods?

3. Admission Control
OPA: has pod all the required labels ?

```

<details>
<summary>What is OPA?</summary>
Open Policy Agent - open source, general purpose policy engine


* not k8s specific
* Uses Rego Language
  * works with JSON/ YAML
* Used as an  Admission Controllers in k8s 

```
OPA GateKeeper  --> OPA Policy Agent
(k8s CRDs)
```

</details>

<details>
<summary>What CRDs does the OPA provide?</summary>

```yaml

# This is a Constriant Template
apiVersion: templates.gatekeeper.sh/v1beta1
kind: COnstraintTemplate
metadata:
  name: k8srequiredlabels
```

# This is a Constraint that states pods must have label X
```yaml
apiVersion: constraints.gatekeeper.sh/vibeta1
kind: K8sRequiredLabels
metadata:
  name: pod-must-have-gk
```
</details>

---

### Lab: Install OPA Gatekeeper
<details>
<summary>How to install OPA GateKeeper</summary>

```
1. Make sure no ohter Admission Plugins enabled, other than NodeAdmission

2. kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/release-3.1/deploy/gatekeeper.yaml

3. Observe new namespace: gatekeeper-system

4. Observe installation added a validating webhook
validatingwebhookconfiguration.admissionregistration.k8s.io/gatekeeper-validating-webhook-configuration created


```
</details>

---

<details>
<summary>What are admission webhooks?</summary>

```
2 - validating admission webhook
    - invoked to enforce and reject requests based on custom policies
    mutating admission webhook
    - invoked first, modifying objects sent to the API to enforce custom defaults

```
</details>

---

### Lab: Create a Deny All OPA Policy
<details>
<summary>Create a Custom Deny All Policy</summary>

```
1. view the CRDs
kubectl get crd

2. Create a ConstraintTemplate

3. Verify
kubectl get constrainttemplates

4. Create a Constraint

5. Run a pod
kubectl run pod --image=nginx

# vid 103
```

</details>

---

### Lab: Create Custom Policy Set NameSpace Labels

<details>
<summary>Create a Policy - All namespaces must have label cks</summary>

```
# ConstraintTemplate will have rego
1. Create a Constraint Template

2. Create Contraint

# View CustomResourceDefinitions
3. kubectl get crd

# Create a new Contraint
4. 

5. Get described Constrint
kubectl describe k8srequiredlabels ns-must-have-cks


```


</details>


* Sample policies
  * Images must originate from 
  * Pods must have CPU Limits Set
  
* OPA Testing
  * pa test image-registry-check-temp.rego test-image-registry-check.rego
  * 

**NOTE:** Relevant Talk
* Policing Your Kubernetes Clusters with open Policy Agent (OPA)
