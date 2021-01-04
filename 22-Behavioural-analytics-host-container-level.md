# Behavioural Analytics - host and contaienr level

* syscall and processes
* strace and /proc
* Tools and scenarios - Falco

---

## Lab: Strace

* strace - intercepts / logs syscalls 

<details>
<summary>do an strace of ls and observe</summary>
</details>

---

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

---

## Lab: Get Env Varaibles from /proc
<details>
<summary>Read the Secret stored as an environment variable from the host filessytem</summary>

```
# Find which node is running the pod
1. kubectl get pod -A -owide 

# get PID of process
2. ps -aux | grep <pod process>

# dump environment
3. strings /proc/<pid>/environ
```
</details>

---

## Lab: Falco

* Cloud Native runtime security
* Utilizes kernel tracing built in to linux
* Describe security rules against a system
* Describe unwanted behaviour

<details>
<summary>Install Falco on Node</summary>
```
Install as a standalone
```
</details>

## Lab: Observe Falco Violations

<details>
<summary>Observe violations when you exec processes in the container</summary>
</details>

<details>
<summary>Observice violations when you append to /etc/passwd</summary>
</details>

## Lab: Flaco - whera re rules?

<details>
<summary>Where are teh falco rules</summary>

```
/etc/falco/falco_rules.yaml
```
</details>

## Lab: Change a rule in Falco
<details>
<summary>Change Output format of Falco Rule</summary>

```
Rule: A shell was spawned in a container with an attached terminal

Output Format: TIME,USER-NAME,CONTAINER-NAME,CONTAINER-ID
Priority: Warning

cd /etc/falco

grep 'A shell was spawned in a container' /etc/falco/*

Copy rule Terminal shell in container into falco_rules_local.yaml

Override 
```

</details>

## Relevant talks

* What have syscalls done for you lately - Liz Rice
* Intro : Falco Kubecon 2018
