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

## Recomended
* WinSCP or similar software on your local computer - will be used to edit template file in an graphical editor (Notepad++ for example) instead of your console

## Application
We will be using Pigallery2 application as an example

* [Pigallery2 docker homepage](https://hub.docker.com/r/bpatrik/pigallery2 "Pigallery2 docker homepage")
* [Pigallery2 homepage](http://bpatrik.github.io/pigallery2/ "Pigallery2 homepage")


## Step 1 - Create a pod
[Kubernetes Pods and Containers](https://kubernetes.io/docs/tasks/configure-pod-container/ "Kubernetes Pods and Containers")

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

## Step 2 - create service
Service will expose the pod port.

[Kubernetes services](https://kubernetes.io/docs/concepts/services-networking/service/ "Kubernetes services")



Generate service template:

```sh
kubectl create service nodeport pigallery2 --tcp=80 --node-port=30080 --dry-run=client -o yaml > step2-service.yaml
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
````sh
kubectl cp images pigallery2:/app/data/images
````

Check the browser again.

## Step 3 - create deployment
Service will expose the pod port.

[Kubernetes services](https://kubernetes.io/docs/concepts/services-networking/service/ "Kubernetes services")

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



