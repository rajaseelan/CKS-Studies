# Supply Chain Security - Static Analysis of User Workloads

* looks at the source code and text files
* checks agains rules
* enforce rules

Examples of Static Analysis rules:
* Always define resource requests and limits
* Pods should never use the default ServiceAccount
* Static Analysis happens when the pod is being built.
* the rules are checked again during deployment via PSP / OPA

## kubesec

* from [kubesec.io](kubesec.io)
* Open Source

## Lab: kubesec

<details>
<summary>Scan a pod manifest using kubesec.</summary>

1. Create a Pod manifest
```
kubectl run nginx --image=nginx --dry-run=client -oyaml > nginx.yaml
```

2. Scan it using kubesec
```
docker run -i kubesec/kubesec:512c5e0 scan /dev/stdin < nginx.yaml
```
</details>

## Lab: Use ConfTest from OPA

<details>
<summary>Static Test pod yaml using OPA and its rego language</summary>

```rego
# pod yaml must container runAsNonRoot: true

package main

deny[msg] {
    input.kind = "Deployment"
    not input.spec.template.spec.securityContext.runAsNonRoot = true
    msg = Containers must not run as root
}
```

Run conftest  
docker run --rm -v $(pwd):/project instrumenta/conftest test deploy.yaml 

</details>

## Lab: confTest for DockerFile

<details>
<summary>Test Dockerfile using OPA ConfTest</summary>

```Dockerfile
# This is the docker file
FROM ubuntu
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y golang-go
COPY app.go .
RUN go build app.go
CMD ["./app"]
```

Rego stanza:  
```rego
package main

denylist = [
  "ubuntu"
]

deny[msg] {
    input[i].Cmd == "from"
    val := input[i].Value
    contains(val[i], denyList[_])

    msg = sprintf("unallowed image found %s", [val])
}
```
Run conftest on this:  
```

```
</details>
