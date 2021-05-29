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
kubectl -n mariadb describe pod -l app=mariadb
```

Check the service:
```sh
kubectl -n mariadb describe service -l app=mariadb
```

Connect to the database:

```sh
kubectl -n mariadb exec -it <mariadb-pod> -- mysql -u pigallery2 -ppigallery2PWD -D pigallery2DB
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