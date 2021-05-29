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
