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

### Lab: Steal Secrets from Docker Container
<details>
<summary>Possible ways to get secrets</summary>

```
# secrets stored in environment variables
1. docker inspect <container>

# secrets mounted as volumes
# copy out files and view
docker cp container:/path/to/mounted/secret /localpath
```
</details>

### Lab: Get Secrets from etcd
<details>
<summary>Possible ways to get secrets from ETCD</summary>  

```
# get the crets kube-apiserver uses to talk to etcd
1. grep etcd /etc/kubernetes/manifests/kube-apiserver.yaml

# test connection to etcd
ETCDCTL_API=3 etcdctl --cert --key --cacert endpoint health

# get secret 
#
# Format /registry/secrets/<namespace>/<secret-name>
#
ETCDCTL_API=3 etcdctl --cert --key --cacert get /registry/secrets/default/secret2
```
</details>

## Encrypt ETCD

1. Create an `EncryptionConfiguration`
1. Pass toapiserver via argument `--encryption-provider-config=/path/to/encryption-config.yaml`

### EncryptionConfiguration format


#### EncryptionConfiguration Example #1
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources: 
  - resources:
    - secret # k8s objects in etcd that need to be encrypted
    providers:
    - identitity: {} # default provider, plaintext
    - aesgcm:
        keys:
        - name: key1
          secret: XXXXXXXXX
        -name: key2
          secret: XXXXXXXXX
    - aescbc:
        keys:
        - name: key1
          secret: XXXXXXXX
        - name: key2
          secret: XXXXXXXX

```
* Encryption providers used **in order**
* First one (`identity`) used for encryption on save (unencrypted)
* To **Read**, there are **options**
  * Unencrypted (`identity`)
  * `aesgcm` encryption
  * `aescbc` encryption

#### EncryptionConfiguration example #2
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources: 
  - resources:
    - secret # k8s objects in etcd that need to be encrypted
    providers:
    - aesgcm:
        keys:
        - name: key1
          secret: XXXXXXXXX
        -name: key2
          secret: XXXXXXXXX
    - identitity: {} # default provider, plaintext
```
* Encryption proiders used **in order**
* Saved in encrypted form using `aesgcm` algo
* To **Read**
  * Decrypt from `aesgcm`
  * Decrypt in plaintext  

**Important** PlainText Provider (`identity`) *required*, else unencrypted secrets *cannot* be read by kube-apiserver.

### How to encrypt all secrets in ETCD?
<details>
<summary>Encrypt all secrets in ETCD</summary>

```
kubectl get secrets --all-namespaces -o json | kubectl replace -f -
```

</details>

### How to decrypt all secrets in ETCD
<details>
<summary>Decrypt All secrets in ETCD</summary>


```yaml
# 1. create a new EncryptionCOnfiguration

apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
    - secrets
    providers:
    - identity: {}
    - aescbc:
        keys:
          - name: key1
            secret: <XXXX Base64 encoded secret>
```

```
# 2. re-read / store secrets as plaintext
kubectl get secrets --all--namespaces -o json | kubectl replace -f -
```

</details>