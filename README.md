# Kubernetes - hands-on session

## Prerequisites

* Linux with installed Kubernetes (VM, physical machine, does not matter)
  * see [microk8s in Ubuntu](Microk8s-in-Ubuntu-Server.docx) as an inspiration
* Command line console with connection to the Kubernetes (putty, WSL, cmd... does not matter)
* k8s Github repository cloned in the VM

```sh
git clone https://github.com/belkop-ghb/k8s.git
```

Enter the cloned repository. Scripts will use files of this repository.

```sh
cd k8s
```

## Recomended

* WinSCP or similar software on your local computer - will be used to edit template file in a graphical editor (Notepad++ for example) instead of linux console

* Two displays - one for watching, second for working

## Application

We will be using Pigallery2 application as an example

* [Pigallery2 docker homepage](https://hub.docker.com/r/bpatrik/pigallery2 "Pigallery2 docker homepage")
* [Pigallery2 homepage](http://bpatrik.github.io/pigallery2/ "Pigallery2 homepage")

## Node

Node is already installed - microk8s or other kubernetes implementation.

[Kubernetes node](https://kubernetes.io/docs/concepts/architecture/nodes/ "Kubernetes node")

```sh
kubectl get nodes -o wide
```

You should see something similar in you console:

```sh
peter@ubuntu:~$ kubectl get nodes -o wide
NAME     STATUS   ROLES    AGE    VERSION                     INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
ubuntu   Ready    <none>   3d8h   v1.20.7-34+df7df22a741dbc   172.27.216.204   <none>        Ubuntu 18.04.5 LTS   4.15.0-143-generic   containerd://1.3.7
```

## [Session1](session1)

* Pod
* Service
* Deployment
* Environment variables
* Configmap
* Volume
* Persistent volume
* Probes

## [Session2 - in preparation](session2)
