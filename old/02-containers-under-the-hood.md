# Containers under the hood

## Containers and Images

### Building Images

Dockerfile -> build-> image -> (docker run) -> Container  

A Process which runs on the Linux Kernel, but cannot see everything (namespaced) 


## Namespaces and cgroups

### Linux Kernel Namespaces

Type | Description
--- | ---
PID | Isolates processes
Mount | restrict access tomounts
Network | restrict network access
User | Different set of user ids used

### cgroups

Restrict resource usage.  
* RAM
* Disk
* CPU

### Run containers in same namespace

1. Create first container
`docker run -d --name c1 ubuntu sh -c 'sleep 1d'`

1. Create 2nd container in same PID Space `docker run -d --pid=container:c1 --name c2 ubuntu sh -c 'sleep 99d'`

1. Execute a ps from within container: `docker exec c2 ps aux`
```
root@cks-master:~# docker exec c2 ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0   2612   544 ?        Ss   20:19   0:00 sh -c sleep 1d
root         6  0.0  0.0   2512   588 ?        S    20:19   0:00 sleep 1d
root        13  0.3  0.0   2612   608 ?        Ss   20:22   0:00 sh -c sleep 999d
root        18  0.0  0.0   2512   588 ?        S    20:22   0:00 sleep 999d
root        19  0.0  0.0   5900  2888 ?        Rs   20:22   0:00 ps aux
```


