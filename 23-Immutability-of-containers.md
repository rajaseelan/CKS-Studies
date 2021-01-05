# Immutability of container

* Container won't be modified in its lifetime.
* An Immutable container's lifecycle:
  * You cannot modify, so changes require a redeployment

How to enfore immutability?  
* Dockerfile
  * remove bash shell
  * make filesystem readonly
  * run as non-root

* pre-built Container - modifying for immutability
  * Pod startup cycle
    * Start -> App Container -> Script chmod 555 -RR /
  * StartupProbe
    * first script that runs
    * use to set rootfs as readonly
  * SecurityContents
    * Must run as readonly filesystem
  * PodSecurityPolicies
  * initContainer
    * Shared volume. init container creates / pulls data sources
    * app container just servers output

## Lab: Use startupProbe to remove touch and bash from container

<details>
<summary>Remove touch & bash at startup using a startupProbe</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: immutable
  name: immutable
spec:
  containers:
  - image: httpd
    name: immutable
    resources: {}
    startupProbe:
      exec:
        command:
          - rm
          - /bin/bash
      initialDelaySeconds: 5
      periodSeconds: 5
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
</details>

## Lab: PodSecurityContext to make filesystem readonly
<details>
<summary>Readonly Filesystem in Pod Spec Using SecurotiyContext</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: immutable
spec:
  containers:
  - name: httpd
    image: httpd
    securityContext:
      readOnlyRootFilesystem: true
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status

```

</details>

<details>
<summary>Create a read only docker container with a temp writable directory</summary>

```
docker run --read-only --tmpfs /run my-container
```
</details>

## Lab: Enable Audit Logging in Apiserver
<details>
<summary>Configure apiserver to store audit logs json</summary>

```
1. create an audit folder
/etc/kubernetes/audit/

2. Create a policy file
# /etc/kubernetes/audit/policy.yaml
# 
# Log Everything on metadata level
kind: Policy
rules:
- level: Metadata

3. Edit kube-apiserver manifest
--audit-policy-file=
--audit-log-path=
--audit-log-maxsize=
--audit-log-maxbackup=5

```
</details>

## Lab: Create k8s secret, observer JSON audit logs
<details>
<summary>Create k8s secret, observer JSON audit logs</summary>

```
k create secret geneic my-secret --from-literal=user=admin

grep admin audit.log
```
</details>

## Lab: Create Complex Audit Log Policy
<details>
<summary>Nothing from stage RequestReceived, get, watch, list. Secrets - only metadata level, Everything else - RequestResponse level</summary>

1. Create yaml (use docs)

2. 
</details>

## Lab: Investigate API access History
<details>
<summary>Investigate API access History</summary>

```
Instructions
1. Change Policy to include Request + Response from Secrets
2. Create a new ServiceAccount + Secret, confirm RequestResponse are available
3. Create a Pod that uses the ServiceAccount
```
</details>

Relevant Talks:  
* Auditing in Kuberentes 101 - Nikhita Raghunath, Loodse