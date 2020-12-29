# Cluster Setup - Verify Platform Binaries

# Hashes
* SHA / MD5 algorithms

# Verify binaries
Sample exercise:  
Download & Veryfy k8s binaries.  

# Verify Binaries in container

Verify hash of running binary with downloaded version.  

1. Find the kube-apiserver container
```
docker ps | grep kube-apiserver
```

2. Copy root fs of container into local dir
```
docker cp <pod_hash>:/ path_to_random_directory/
```

3. sha512 the kube-apiserver binary
```
sha512sum path_to_random_directory/usr/local/bin/kube-apiserver
```

4. Compare with hash from website.