# Behavioural Analytics - host and contaienr level

* syscall and processes
* strace and /proc
* Tools and scenarios - Falco

## Lab: Strace

* strace - intercepts / logs syscalls 

<details>
<summary>do an strace of ls and observe</summary>
</details>

## Lab: /proc directory

* use strace to try and interrogate etcd
* use the /proc where possible

```
# find etc docker containr
1. docker ps | gtrep etcd

2. ps aux | grep etcd

3. Get pid

4. strace the command
strace -f -p 3502

5. strace and get a summary of stats
strace -f -cw -p 3502

6. Observe etcd in /proc dir

7. Add a secret.

8. grep it out of the /proc using the database file descriptor

```

## Lab: Get Env Varaibles from /proc
<details>
<summary>Read teh Secret stored as an environment variable from the host filessytem</summary>

```
# Find which node is running the pod
1. kubectl get pod -A -owide 

# get PID of process
2. ps -aux | grep <pod process>

# dump environment
3. strings /proc/<pid>/environ
```
</details>