# Kubernetes FAQ

## What is Kubernetes (K8s)?
Kubernetes, also called **K8s**, is a system that helps manage and run containers automatically. It makes sure your apps stay running, can scale up or down, and recover if something goes wrong.

## How are Kubernetes and Docker linked?
Docker is used to create and run containers, while Kubernetes helps manage many containers across multiple machines. Kubernetes can use Docker to run applications but can also work with other container systems like **containerd**.

## What is the role of etcd in K8s?

```
etcd holds the current status of any k8s component. Let say the desired state in my manifest has 2 nginx replicas and status of current k8s cluster is having 1 then it will compare the desired and current status and do fixing. 

cluster changes get stored in the key value store. 
```