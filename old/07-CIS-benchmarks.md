# CIS Benchmarks

* Center for Internaet Security
* best practices for secure configuration of a target system

## CIS Recommendations
* provides default security rules

# Sample auditing
1. Install kube-bench
```
# Choose the steps for run inside container

docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro -t aquasec/kube-bench:latest [master|node] --version 1.13

```

Refer to PDF for Failed benchmarks are suggested corrective actions.
