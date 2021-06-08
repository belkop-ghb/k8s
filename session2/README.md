# Session 2 - improve sample application

It is not necessary to complete the whole session 1 to work on session 2.
To get the status of the Session 1, you only need to perform these steps:

* Last step of session 1:

```sh
kubectl apply -f session1/step8-probes.yaml
```

* Copy images to the appropriate location:

```sh
sudo cp -r images/ /opt/
```

Check the application in browser:

http://\<VM-IP>:30080

## Prerequisite

In order to use DNS names we must enable DNS in microk8s.
Check, if your DNS is enabled.

```sh
microk8s status
```

If the DNS is not enabled, enable it:

```sh
microk8s enable dns
```

If the metrics-server is not enabled, enable it:

```sh
microk8s enable metrics-server
```

## Step1 - Database

We will create a database template using knowledge earned in [session1](../session1).

See details about official MariaDB image:
[MariaDB image](https://hub.docker.com/_/mariadb "MariaDB image")

See details about kubernetes secrets:
[Kubernetes secrets](https://kubernetes.io/docs/concepts/configuration/secret/ "Kubernetes secrets")

* See the template: [step1-db.yaml](step1-db.yaml)
* See non-encoded secrets: [secret.properties](secret.properties)

Secret encoding:

```sh
echo -n "superStrongPassword" | base64
```

Secret decoding:

```sh
echo -n "c3VwZXJTdHJvbmdQYXNzd29yZA==" | base64 --decode
```

Apply the template and create the database pod with whole infrastructure:

```sh
kubectl apply -f session2/step1-db.yaml
```

Check the pod:

```sh
kubectl describe pod -l app=mariadb
```

Check the service:
```sh
kubectl describe service -l app=mariadb
```

Connect to the database:

```sh
kubectl exec -it <mariadb-pod> -- mysql -u pigallery2 -ppigallery2PWD -D pigallery2DB
```

You should see:

```sh
peter@ubuntu:~/k8s$ kubectl exec -it mariadb-64bdbb9bd8-9fjl9 -- mysql -u pigallery2 -ppigallery2PWD -D pigallery2DB
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 352
Server version: 10.6.1-MariaDB-1:10.6.1+maria~focal mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.     

MariaDB [pigallery2DB]>
```

## Step 2 - prepare sample application to use the database

We have copied the last template of session1 as a starting point to next steps:

*cp session1/step8-probes.yaml session2/step2-app.yaml*

Changes have been performed on the template *step2-app.yaml*:

* Readiness and liveness probes
* ConfigMap - login disabled
* Pod - config volume

Check the template: [step2-app.yaml](step2-app.yaml)

Add folder for the application configuration:

```sh
sudo mkdir /opt/pigallery2
```

Apply the application template:

```sh
kubectl apply -f session2/step2-app.yaml
```

## Step 3 - adapt application to use database

See App documentation how to correctly configure the secret keys:
[Pigallery2 manual page](https://github.com/bpatrik/pigallery2/blob/master/MANPAGE.md "Pigallery2 manual page")

Check the template: [step3-app.yaml](step3-app-db.yaml)

* Secret
* ConfigMap
* Pod

Apply the template:

```sh
kubectl apply -f session2/step3-app-db.yaml
```

Connect to the database:

```sh
kubectl exec -it <mariadb-pod> -- mysql -u pigallery2 -ppigallery2PWD -D pigallery2DB
```

In the database, check the data:

```sql
select id, name from media_entity;
```

Check the connectivity between app *pod* and DB *service*:

```sh
kubectl exec -it <podName> -- wget --spider --timeout=3 mariadb-svc.default.svc.cluster.local:3306
```

If you see something like : *HTTP request sent, awaiting response... 200 No headers, assuming HTTP/0.9*, connection is successful.

## Step 4 - create productive namespace

Let's say, that we have more instances of the same application.
We have to separate these two instances - into two namespaces (default / productive).

Check and apply the template: [step4-namespace.yaml](step4-namespace.yaml)

```sh
kubectl apply -f session2/step4-namespace.yaml
```

## Step 5 - create productive database

Check and apply the template: [step5-db-prod.yaml](step5-db-prod.yaml)

```sh
kubectl apply -f session2/step5-db-prod.yaml
```

Check the pod is working:

```sh
kubectl get pods -n production
```

## Step 6 - create productive app

Check and apply the template: [step6-app-prod.yaml](step6-app-prod.yaml)

```sh
sudo cp -r /opt/pigallery2/ /opt/pigallery2-prod
```

```sh
kubectl apply -f session2/step6-app-prod.yaml
```

Check the pod is working:

```sh
kubectl get pods -n production
```

Check, that the productive instance is running:

*http://<VM's IP>:30081*

Check the connection between DEFAULT pod and PRODUCTION DB:

```sh
kubectl exec -it <podName> -- wget --spider --timeout=3 mariadb-svc.production.svc.cluster.local:3306
```

Switch namespace in the context:

```sh
kubectl config set-context --current --namespace=production
```

Check the connection between PRODUCTION pod and PRODUCTION DB:

```sh
kubectl exec -it <podName> -- wget --spider --timeout=3 mariadb-svc.production.svc.cluster.local:3306
```

## Step 7 - network policy

[Kubernetes network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/ "Kubernetes network policies")

Check and apply the template: [step7-network-policy.yaml](step7-network-policy.yaml)

```sh
kubectl apply -f session2/step7-network-policy.yaml
```

## Step 8 - application pod labels

Check and apply the template: [step8-app-prod.yaml](step8-app-prod.yaml)

```sh
kubectl apply -f session2/step8-app-prod.yaml
```

## Step 9 - resources

[Container resources](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/ "Container resources")

Copy a sample video to the images folder:

```sh
sudo cp videos/earth.mp4 /opt/images/
```

Run a video conversion and watch the load:

```sh
watch -n 1 kubectl top pods
```

Check and apply the template: [step9-app-resources.yaml](step9-app-resources.yaml)

```sh
kubectl apply -f session2/step9-app-resources.yaml
```

Run a video conversion again and watch the load:

```sh
watch -n 1 kubectl top pods
```

## Common troubleshooting

Logs of the pod:

```sh
kubectl logs -f <pod_name>
```

Events in the namespace:

```sh
kubectl get events
```

Get all objects:

```sh
kubectl get all --all-namespaces
```

Delete all objects of the template:

```sh
kubectl delete -f <templateName.yaml>
```
