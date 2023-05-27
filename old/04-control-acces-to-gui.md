# Access to GUI Elements

## connecting securely

`kubectl proxy` 
* proxy between kube-apiserver and local machine
* uses permissions of `kubeconfig` file

`kubectl port-forward`  
* forward connection (tcp) from pod port

## Installing Dashboard

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.1.0/aio/deploy/recommended.yaml
```

## Configuring insecure access to dashboard

** If you're doing this on a publicly accesible image - DON'T**

1. Edit the deployment yaml for `service/kubernetes-dashboard`
```
# add insecure-port
  - insecure-port=9090

# remove generate https certs option
```

2. Convert clusterip service to nodeport and change http port
```
k -n kubernetes-dashboard edit svc kubernetes-dashboard 

 22   ports:
 23   - nodePort: 30033
 24     port: 9090
 25     protocol: TCP
 26     targetPort: 9090
 27   selector:
 28     k8s-app: kubernetes-dashboard
 29   sessionAffinity: None
 30   type: NodePort

```

3. Executing a get service will show you the dashboard and the nodeport created, accesible by public ip.  
```
k -n kubernetes-dashboard get svc 
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
dashboard-metrics-scraper   ClusterIP   10.104.236.88   <none>        8000/TCP         11m
kubernetes-dashboard        NodePort    10.106.56.32    <none>        9090:30033/TCP   11m
```

## Create a proper dashboard service account

Get the name of the public viewer

# configure 
* default viewer role - namespace scoped
* clusterwide viewer role - clusterrole
* auth for dashboard

```
certifik8s
Liz Rice - What have namespaces done for you lately?


https://www.youtube.com/watch?v=wqsUfvRyYpw

student@cks-master:~$ k run frontend --image=nginx
pod/frontend created
student@cks-master:~$ k run backend --image=nginx 
pod/backend created
student@cks-master:~$ k expose pod frontend --port 80
service/frontend exposed
student@cks-master:~$ k expose pod backend --port 80 
service/backend exposed
student@cks-master:~$ k get pod,svc

k exec -it backend curl frontend 
k exec -it frontend  --  curl backend

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

# create policy 
# frontend outgoing traffic to backend
# backed outgoing traffic to foront end 


# backend to cassandra
 create namesapce cassandra
 label cassandra ns=cassandra
 enabel backend to cassandra
 create default deny for cassandra
 create rule from backend to cassandra after default deny
 
 
 ```