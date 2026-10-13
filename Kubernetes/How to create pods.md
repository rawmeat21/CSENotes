Let's create a pod which will run an instance of the image `nginx`:

```bash
$ kubectl create deployment mynginx (<image name>) --image=nginx:latest (<name on docker registry>)
```

```bash
❯ kubectl get deployments                               
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
mynginx   0/1     1            0           29s
```

To see all pods:

```bash
❯ kubectl get pods       
NAME                       READY   STATUS    RESTARTS   AGE
mynginx-79b9585778-8dn94   1/1     Running   0          64s   <---pod
```

`nginx` listens on port 80. But, right now it's running inside a container. So, we need to expose a port from outside. This is done using a service object in Kubernetes:

```bash
kubectl expose deployment mynginx --port=80 --type=LoadBalancer
```

Check:

```bash
❯ kubectl get services                                           
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP        22m
mynginx      LoadBalancer   10.108.31.208   <pending>     80:30142/TCP   14s
```

If you tried `localhost:30142`, it wouldn't work. Here's why:

`kubectl` is just a tool that sends requests (“list the pods”, “make a deployment”) to a Kubernetes cluster over the network. 

It has no cluster of its own. It needs to know _which_ cluster to talk to, and that’s stored in a file on your computer: `~/.kube/config`.

When you ran `minikube start`, minikube did two things:

1. Created a small Kubernetes cluster inside a Docker container.
2. Wrote an entry into `~/.kube/config` saying “there’s a cluster at this address, use it”, and made it the default.

see:

```bash
❯ kubectl config current-context
minikube
```

Get IP of minikube cluster:

```bash
❯ minikube ip
192.168.49.2
```

Then go to `192.168.49.2:30142`.


Or try doing:

```
❯ minikube service mynginx
┌───────────┬─────────┬─────────────┬───────────────────────────┐
│ NAMESPACE │  NAME   │ TARGET PORT │            URL            │
├───────────┼─────────┼─────────────┼───────────────────────────┤
│ default   │ mynginx │ 80          │ http://192.168.49.2:30142 │
└───────────┴─────────┴─────────────┴───────────────────────────┘
🎉  Opening service default/mynginx in default browser...
```

Does the same thing.

![](../assets/Pasted%20image%2020261013194605.png)

minicube makes 1 cluster. 






