# Node Metadata Protection

* deny pod access to metadata services using firewalls / cloud service provider facilities.

e.g 
GCP Metadata  
```
curl "http://metadata.google.internal/computeMetadata/v1/instance/disks/ -H "Metadata-Flavor: Google"
```

## Protect GCP Metadata by Create defauly deny policy
```
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0
        except:
        - 169.254.169.254/32 #GCP Compute Engine Metadata server
```
