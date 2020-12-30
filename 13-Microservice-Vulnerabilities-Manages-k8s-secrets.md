# Microservice Vulnerabilities Manage k8s secrets

* secrets are used by k8s to decouple it from the CI/CD workflow. Credentials / auth tokens do not need to be embedded into the source code repository.

## Lab: Inject secrets into pod

<details>
<summary>Create 2 secrets using imperative commands</summary>

```
kubectl create secret generic mount-secret --from-literal=my-name=tom

kubectl create secret generic env-secret --from-literal=env_type=production
```

</details>

<details>
<summary>Inject into pod, as a volumount mount & env variable</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secrets-pod
  name: secrets-pod
spec:
  volumes:
  - name: secret-vol
    secret:
      secretName: mount-secret
  containers:
  - image: nginx
    name: secrets-pod
    env:
      - name: env_type
        valueFrom:
          secretKeyRef:
            name: env-secret
            key: env_type
    volumeMounts:
    - name: secret-vol
      mountPath: /etc/secrets
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

</details>

