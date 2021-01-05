# Kernel hardening

* AppAmor (Docker & K8s)
* Seccomp (Docker & k8s)

## How Does Container Isolation work?
1. Isolate by namespaces  
    Processes
    Users
    Filesystem

1. Cgroups  
    Resource useage restrictions

1. every proc can talk to the kernel directly
     Enter seccomp - a layer between processes and kernel
     Seccomp sits between userspace and Syscall Interface

     Applications  
     Libraries  
     --- seccomp ---  
     syscall interface
     Linux Kernel
     Hardware




# AppAmor

* Like SELinux
* Create Profiles for Apps
  * allow disallow access to items like filesystem / processes / network
* Controlled by creating Profiles for app
  * i.e create a appamor profile for the kubelet

## Appamor Profiles
Profile | Capabilities
--- | ---
Unconfined  | Processes can escape - free for all
Complain    | Processes can escape, but violations logged
Enforce     | Processes canot escape

## Appamor commands

    # show all profiles
    aa-status

    # generate a new profile
    aa-genprof

    # put profile in enforce mode
    aa-enforce

    # update the profile if app produced more usage logs
    aa-logprof


# Lab: Setting up appamor for curl
<details>
<summary>Create a simple apparmor profile for curl</summary>

    # be on worker node
    #
    # run aa-status
    #
    # observe profiles loaded vs enfore
    # 
    ...snip...
    apparmor module is loaded.
    16 profiles are loaded.
    16 profiles are in enforce mode.
    /sbin/dhclient
    /usr/bin/lxc-start
    /usr/bin/man
    ...snip...

    # Install apparmor tools 
    apt install appamour-utils

    # Generate a new apparmor profile
    aa-genprof curl

    # finish profiling immediately
    
    # you can execute curl, but cannot access a website

    # list loaded apparmor profiles
    aa-status

    # Observe that the curl profile is loaded and in enforce mode

    # locate profile on hard drive
    /etc/apparmour.d/usr.bin.curl

    # update profile of curl
    aa-logprof

    # apparmor will read the violations in syslog
    # display and prompt to update the curl apparmor profile
</details>

## Lab: Run docker container with apparmor profile
<details>
<summary>Run docker container with apparmor profile</summary>

    docker run --security-opt apparmor=docker-default -d nginx 

</details>

---

## Requirements for running apparmor on k8s

* Container runtime must support AppArmor
* AppArmor needs to be installed on every node
* AppArmor profiles need to be available on every node
* AppAormor profiles are specifed per container
  * Done using annotations

## Lab: apparmor for k8s
<details>
<summary> Use apparmor in k8s </summary>

    # Assuptions - pre-reqa are met

    apiVersion: v1
    kind: Pod
    metadata:
      annotations:
        # note the container name and profile on localhost
        container.apparmor.security.beta.kubernetes.io/<container name> : localhost/<profile_name>
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
      

</details>



# Seccomp
