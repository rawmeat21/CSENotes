![](../assets/Pasted%20image%2020261013194838.png)

The flow is:

1. Create a dockerfile for your application and create a docker image.
2. Publish it to Docker registry.

![](../assets/Pasted%20image%2020261013195358.png)

3. Then:

```bash
$ kubectl create deployment my-project --image=<your image name on docker registry>
```

4. Do additional things (exposing ports)


When you update your app:

1. Create new docker image
2. Publish changes to docker registry
3. Then:

Make deployment point to a new image:
```bash
$ kubectl set image deployment my-project <container name>=<new image name>
```
Notice one thing:

![](../assets/Pasted%20image%2020261013200745.png)

The old one is still there (running) in first output. Kubernetes doesn't take away the old container until the new one pops up. This prevents downtime situations.

Next, the new one is running (updated) and old one is terminating.

You may want to check the status of update (rollout):

```bash
kubectl rollout status deployment my-project
```

### Rollback

Rollback is the reverse of update, you revert to a previous version.

```bash
kube rollout undo deployment my-project
```


### Self healing

How does kubernetes handle app crashes?

The rule is simple: K8s will restart it always!

So, even if your app dies, it will get restarted automatically




