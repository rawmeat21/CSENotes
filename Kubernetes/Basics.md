All pictures here are taken from this course: https://youtu.be/rBeyHDKLVqM?si=q3BCs6hxFTNdzoYJ

![[Pasted image 20260913141521.png]]

Kubernetes does container orchestration, which means it manages containers.

Why use kubernetes?

![[Pasted image 20260913141805.png]]

You have your website deployed to a server. 

![[Pasted image 20260913141913.png]]

What happens on heavy load?

![[Pasted image 20260913141929.png]]

What if the server fails?


Solution?

![[Pasted image 20260913142006.png]]

deploy on multiple servers.

How to control and manage these nodes? 

Yep, that's what Kubernetes does.


### Architecture of Kubernetes

![[Pasted image 20260913142214.png]]

Master - Controller
Worker nodes - Our servers, which hold containers

![[Pasted image 20260913142330.png]]

![[Pasted image 20260913142354.png]]

A group of nodes is called a cluster. 

![[Pasted image 20260913142544.png]]

The Master controls these worker nodes.
It has an API server.
Every worker node has an agent called kubelet. 

The Master can be on a separate server or on the same server as some other worker node. 

### Components of K8s

![[Pasted image 20260913142648.png]]


### Pod

![[Pasted image 20260913142731.png]]

Basically, servers run pods, and pods run containers. Pod is an isolated environment with it's own resources. Pod is the smallest unit in K8s.

#### Features of a pod

**1. Shared network namespace**  
Every container in a pod shares one IP address and one port space. They talk to each other via `localhost`. From outside the pod, all containers inside look like a single network endpoint.

**2. Shared storage volumes**  
You can define a volume (shared folder) at the pod level, and mount it into multiple containers inside that pod. 

**3. Co-scheduling (they run on the same node, always)**  
All containers in a pod are guaranteed to land on the same physical/virtual machine. You can't have half a pod on one node and half on another. This is precisely why pods aren't meant for loosely-related things.

**4. Shared lifecycle**  
Containers in a pod start and stop together as a unit. If the pod is deleted, every container in it is deleted. They don't have independent lifecycles from each other.

**5. Init containers**  
A pod can define containers that run to completion _before_ the main containers start — used for setup tasks (fetching a config file, waiting for a dependency to be ready). Once they finish successfully, they're done; they don't run alongside the main containers.

**6. A single, ephemeral IP address**  
The pod gets one IP from the cluster's pod network (ex: `10.244.0.0/16`). If the pod dies and gets recreated, even by the same Deployment, it usually gets a _new_ IP. This is why you don't hardcode pod IPs; you use a Service (a stable address that points at whichever pods currently exist) to talk to them instead.

**7. Resource requests/limits, set per-container but tracked at the pod level**  
Each container can declare how much CPU/memory it needs (`requests`) and the max it's allowed to use (`limits`). The _scheduler_ looks at the sum across all containers in the pod when deciding which node has room for it.

**8. Restart policy**  
A pod-level setting controlling what happens when a container inside it crashes: `Always` (default, means always restart), `OnFailure` (only restart if it exited with an error), or `Never`. This is a pod-wide setting, not one you set per container.

**9. Labels and annotations**  
Arbitrary key-value tags you attach to a pod (e.g. `app: nginx`). These aren't just metadata, Services and Deployments use label _selectors_ to find and manage groups of pods. This is the actual mechanism connecting a Deployment to "its" pods; there's no other link between them besides matching labels.

**10. They're disposable by design**  
This is more a philosophy than a "feature," but it's the single most important thing to internalize: you're never supposed to treat an individual pod as precious. Pods get killed and replaced constantly (node fails, you roll out a new version, scheduler decides to rebalance). Anything that needs to survive a pod dying — data, identity, state — has to live outside the pod (a volume backed by real storage, a database, etc.). This is exactly why your `test-nginx` pod from earlier was just gone with no recreation — bare pods have zero resilience by design, and that's the point of contrast with Deployments.

### The Master node

![[Pasted image 20260913143052.png]]

**API server** is the terminal (kubectl).
**Scheduler** is used to **assign nodes to newly created pods.**
**ETCD** is a key-value store, contains all cluster data.
**Control Manager** manages state of the cluster. 

### Worker node

![[Pasted image 20260913143449.png]]

An example of container-runtime is Docker. 


### What can we do with K8s

![[Pasted image 20260913143659.png]]


