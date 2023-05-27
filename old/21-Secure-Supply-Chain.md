# Secure Supply Chain

* What is supply chain?
  * Private Container Registries

* How to use a Private Registry in k8s?
1. Create Pull Secrets
```
kubectl create secret docker-registry my-private-registry \
  --docker-server=my-private-registry-server \
  --docker-username=username \
  --docker-password=password \
  --docker-email=email
```

2. Add registry to ServiceAccount
```
kubectl patch serviceaccount default -p '{ "imagePullSecrets": [ { "name": "my-private-registry" } ]}'
```

## Lab: List all Registries in k8s Cluster


<details>
<summary>List All Regitries in k8s cluster</summary>

```
kubectl get pod -A -oyaml | grep "image: "
```

</details>


<details>
<summary>How do you display the Image Digest?</summary>

```
kubectl -n kube-system get pod kube-apiserver-cks-master -oyaml

# look under 
# status
#   containerStatuses
#     imageID: docker-pullable://k8s.gcr.io/kube-apiserver@sha256:XXXXXXX
```
</details>

## Lab: Whitelist Container Registry with OPA
<details>
<summary>Create a Registry Whitelisting Policy for OPA</summary>

1. create the ContraintTemplate
```yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8sTrustedTemplate
spec:
  crd:
    spec:
      names:
        kind: K8sTrustedImages
  targets:
  - target: admission.k8s.gatekeeper.sh
    rego: |
      package k8strustedimages

      violation[{"msg": msg}]
        image := input.review.object.spec.containers[_].image
        not.startswith(image, "docker.io/")
        not startswith(image, "k8s.gcr.io/")
        msg := "not trusted image"

```

2. Create the constraint
```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sTrustedImages
metadata:
  name: pod-trusted-images
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]

```

3. Try to create a pod
```
# you should get a violation
kubectl run nginx --image=nginx

# The image registry must always be specified
kubectl run nginx --image=docker.io/nginx

```
</details>

## ImagePolicyWebhook

* kube-apiserver - Authenticate -> Authorized -> Admission (Controllers)
* ImagePolicyWebhook contacts an External Service
* External Service revices a "kind":"ImageReview"
  * Allow / Deny based on Criteria

### Lab: Creating an ImagePolicyWebhook

<details>
<summary>use an External Webhook for Image Policy</summary>
1. Enable in kube-apiserver manifest
```
# it is assumed that the OPA is removed 
# in kube-apiserver manifest

--enable-admission-plugins=NodeRestriction,ImagePolicyWebhook
--admission-control-config-file=/etc/kubernetes/admission/admission_config.yaml
```

2. Check /var/log/pods/
# you should see ImagePolicyWebhook no config specified

Example ImagePolicyWebhook


</details>