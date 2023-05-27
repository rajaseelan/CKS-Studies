# Securing Ingress

# What is an ingress
* Focus on the nginx ingress
* Ingress - wrapper around nginx pod
* Ingress points to ClusterIP
* NodePort
  * port on every node's IP
  * points to a ClusterIP
* LoadBalancer -> NodePort -> ClusterIP

# Setup Ingres
For bare metal installs:  
```
https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal
```

Installation line:  
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.40.2/deploy/static/provider/baremetal/deploy.yaml

 
 # test ingress

 # get nodeports
 k get pod,svc # from Ingress Namespace

 # connect via node's external IP
 ```

## Exercise
Create ingress resource with the parameters:
* name: minimal ingress
* path: /service1 to service1 port 80
* path: /service2 to service1 port 80

# Secure Ingress

Creating our own signed certificate + key for Ingress.  

1. Create CSR.
```
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes 

# Fill in the Common Name - secure-ingress.com
```

2. Create a TLS Secret
```
kubectl create secret tls secure-ingress --cert=cert.pem --key=key.pem
```

3. Add HTTPS Capabilities to ingress

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-example-ingress
spec:
  tls:
  - hosts:
      - https-example.foo.com
    secretName: testsecret-tls
  rules:
  - host: secure-ingress.com
    path:
````

4. Test https ingress
```
curl https://secure-ingress.com:30885/service2 -kv --resolve secure-ingress.com:31047:192.168.123.19
```
