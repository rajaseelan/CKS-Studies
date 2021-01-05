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