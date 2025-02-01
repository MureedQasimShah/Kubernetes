# Kubernetes FAQ

## What is Kubernetes (K8s)?
Kubernetes, also called **K8s**, is a system that helps manage and run containers automatically. It makes sure your apps stay running, can scale up or down, and recover if something goes wrong.

## How are Kubernetes and Docker linked?
Docker is used to create and run containers, while Kubernetes helps manage many containers across multiple machines. Kubernetes can use Docker to run applications but can also work with other container systems like **containerd**.

## Difference Between Liveness Probe and Readiness Probe in Kubernetes
1. Liveness Probe

    - Determines if a container is still alive (running and not stuck).
    - If the liveness probe fails, Kubernetes kills the container and restarts it.
    - Useful when an application might get into a deadlock or hang indefinitely.
```bash
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5

```

In the configuration file, you can see that the Pod has a single Container. The **`periodSeconds`** field specifies that the kubelet should perform a liveness probe every 5 seconds. The `initialDelaySeconds` field tells the kubelet that it should wait 5 seconds before performing the first probe. To perform a probe, the kubelet executes the command `cat /tmp/healthy` in the target container. If the command succeeds, it returns 0, and the kubelet considers the container to be alive and healthy. If the command returns a non-zero value, the kubelet kills the container and restarts it.

When the container starts, it executes this command:

```bash
/bin/sh -c "touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600"
```

For the first 30 seconds of the container's life, there is a `/tmp/healthy` file. So during the first 30 seconds, the command `cat /tmp/healthy` returns a success code. After 30 seconds, `cat /tmp/healthy` returns a failure code.

**Create the Pod:**
```bash
kubectl apply -f https://k8s.io/examples/pods/probe/exec-liveness.yaml
```
Within 30 seconds, view the Pod events:

`kubectl describe pod liveness-exec`

The output indicates that no liveness probes have failed yet:

```bash
Type    Reason     Age   From               Message
----    ------     ----  ----               -------
Normal  Scheduled  11s   default-scheduler  Successfully assigned default/liveness-exec to node01
Normal  Pulling    9s    kubelet, node01    Pulling image "registry.k8s.io/busybox"
Normal  Pulled     7s    kubelet, node01    Successfully pulled image "registry.k8s.io/busybox"
Normal  Created    7s    kubelet, node01    Created container liveness
Normal  Started    7s    kubelet, node01    Started container liveness
```

After 35 seconds, view the Pod events again:

`kubectl describe pod liveness-exec`

At the bottom of the output, there are messages indicating that the liveness probes have failed, and the failed containers have been killed and recreated.

```bash
Type     Reason     Age                From               Message
----     ------     ----               ----               -------
Normal   Scheduled  57s                default-scheduler  Successfully assigned default/liveness-exec to node01
Normal   Pulling    55s                kubelet, node01    Pulling image "registry.k8s.io/busybox"
Normal   Pulled     53s                kubelet, node01    Successfully pulled image "registry.k8s.io/busybox"
Normal   Created    53s                kubelet, node01    Created container liveness
Normal   Started    53s                kubelet, node01    Started container liveness
Warning  Unhealthy  10s (x3 over 20s)  kubelet, node01    Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
Normal   Killing    10s                kubelet, node01    Container liveness failed liveness probe, will be restarted
```
Wait another 30 seconds, and verify that the container has been restarted:

`kubectl get pod liveness-exec`

The output shows that RESTARTS has been incremented. Note that the RESTARTS counter increments as soon as a failed container comes back to the running state:

```bash
NAME            READY     STATUS    RESTARTS   AGE
liveness-exec   1/1       Running   1          1m
```
## What is the role of etcd in K8s?

```
etcd holds the current status of any k8s component. Let say the desired state in my manifest has 2 nginx replicas and status of current k8s cluster is having 1 then it will compare the desired and current status and do fixing. 

cluster changes get stored in the key value store. 
```