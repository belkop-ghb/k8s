# Kubernetes - hands-on session

## Prerequisites

* Linux with installed Kubernetes (VM, physical machine, does not matter)
* Command line console with connection to the Kubernetes

You should see something similar in you console:

```sh
user@ubuntu:~$ kubectl get nodes
NAME     STATUS   ROLES    AGE   VERSION
ubuntu   Ready    <none>   22h   v1.20.6-34+e4abae43f6acde
```

* k8s Github repository cloned in the VM
```sh
git clone https://github.com/belkop-ghb/k8s.git
```

Enter the cloned repository. Scripts will use files of this repository.
```sh
cd k8s
```

## Recomended

* WinSCP or similar software on your local computer - will be used to edit template file in an graphical editor (Notepad++ for example) instead of your console

## Application

We will be using Pigallery2 application as an example

* [Pigallery2 docker homepage](https://hub.docker.com/r/bpatrik/pigallery2 "Pigallery2 docker homepage")
* [Pigallery2 homepage](http://bpatrik.github.io/pigallery2/ "Pigallery2 homepage")

## Step 0 - node

Node is already installed - microk8s or other kubernetes implementation.

[Kubernetes node](https://kubernetes.io/docs/concepts/architecture/nodes/ "Kubernetes node")

```sh
kubectl get nodes -o wide
```

## Step 1 - pod

[Kubernetes Pods and Containers](https://kubernetes.io/docs/tasks/configure-pod-container/ "Kubernetes Pods and Containers")

See central docker image repository:
[Docker hub](https://hub.docker.com/ "Docker hub")

### Manual way

One-time create - only for test purpose

```sh
kubectl run pigallery2 --image=bpatrik/pigallery2
```

### Template way

```sh
kubectl run pigallery2 --image=bpatrik/pigallery2 --dry-run=client -o yaml > template.yaml
```

Template has been created for you - see step1-pod.yaml

To apply the template:

```sh
kubectl apply -f step1-pod.yaml
```

## Step 2 - service

Service will expose the pod port.

[Kubernetes services](https://kubernetes.io/docs/concepts/services-networking/service/ "Kubernetes services")

You can generate service template (already generated as step2-service.yaml):

```sh
kubectl create service nodeport pigallery2 --tcp=80 --node-port=30080 --dry-run=client -o yaml > templateService.yaml
```

Use service template:

```sh
kubectl apply -f step2-service.yaml
```

Check the service:

```sh
kubectl describe service pigallery2
```

Check the application in browser:

http://\<VM-IP>:30080

Copy the images to the pod and check it in the browser:

```sh
kubectl cp images pigallery2:/app/data/images
```

Check the browser again.

## Step 3 - deployment

Deployment will cover replicas and container image version.

[Kubernetes deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/ "Kubernetes deployment")

You can create a deployment template (already generated as step3-deployment.yaml):

```sh
kubectl create deployment pigallery2 --image=bpatrik/pigallery2:1.8.2 --replicas=1 -o yaml --dry-run=client > deploymentTemplate.yaml
```

See the step3-deployment.yaml - deployment and service are in one common file:

```sh
cat step3-deployment.yaml
```

Delete all object based on the label:

```sh
kubectl delete all -l app=pigallery2
```

Use the template to create the deployment and service:

```sh
kubectl apply -f step3-deployment.yaml
```

Try to delete the pod:

```sh
kubectl delete pod -l app=pigallery2
```

Deploy new version:

```sh
kubectl set image deployment pigallery2 pigallery2=bpatrik/pigallery2:1.8.5
```

Check changes:

```sh
kubectl describe deployment pigallery2
```

```sh
kubectl describe pod -l app=pigallery2
```

Revert to old version:

```sh
kubectl rollout undo deployment pigallery2
```

## Step 4 - environment variables

[Kubernetes environment variables](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/ "Kubernetes environment variables")

Application manual:
[Pigallery2 manual page](https://github.com/bpatrik/pigallery2/blob/master/MANPAGE.md "Pigallery2 manual page")

```sh
kubectl apply -f step4-env-var.yaml
```

Check the application in browser.

## Step 5 - configmap

[Kubernetes configmap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/ "Kubernetes configmap")

Check and apply the template step5-configmap.yaml

```sh
kubectl apply -f step5-configmap.yaml
```

## Step 6 - volume

[Kubernetes volume](https://kubernetes.io/docs/concepts/storage/volumes/ "Kubernetes volume")

Check and apply the template:

```sh
kubectl apply -f step6-volume.yaml
```

Copy images from git repository to the path configured in the volume:

```sh
sudo cp -r images/ /opt/
```

Check the application in the browser.

Try to delete the pod:

```sh
kubectl delete pod -l app=pigallery2
```

## Step 7 - persistent volume

[Kubernetes persistent volume](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/ "Kubernetes persistent volume")

Check and apply template file:

```sh
kubectl apply -f step7-persistent-volume.yaml
```

Check newly created objects:
```sh
kubectl get pv
```

```sh
kubectl get pvc
```

```sh
kubectl describe pod -l app=pigallery2
```

## Step 8 - probes

[Kubernetes probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/ "Kubernetes probes")

Check and apply template file:

```sh
kubectl apply -f step8-probes.yaml
```

Delete the pod and watch when it will be ready:

```sh
kubectl delete pod -l app=pigallery2
watch -n 2 kubectl get pods
```
