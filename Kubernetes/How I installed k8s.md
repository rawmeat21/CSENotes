#### 1. Disable swap

bash

```bash
sudo swapoff -a
sudo sed -i '/swap/s/^/#/' /etc/fstab
```
The 2nd command comments out the swap line in `/etc/fstab` so it stays off after reboot. Probably don't do this if you need swap space. My vscode lagged like shit after this. 

To turn swap back on:

```bash
sudo swapon -a
```

#### 2. Load kernel modules

bash

```bash
cat << EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

#### 3. Sysctl networking params

bash

```bash
cat << EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

#### 4. Install and configure containerd

bash

```bash
sudo pacman -S containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

Now edit `/etc/containerd/config.toml`. Find this section:


```toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
  SystemdCgroup = false
```



Change it to:

```toml
  SystemdCgroup = true
```



Then:


```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

#### 5. Install kubeadm, kubelet, kubectl (AUR)


```bash
yay -S kubeadm-bin kubelet-bin kubectl-bin
sudo systemctl enable kubelet
```

(kubelet will crash-loop until step 6 — expected, ignore it for now)

#### 6. Initialize the control plane

bash

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

#### 7. Configure kubectl for your user

bash

```bash
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### 8. Install CNI (Calico)

bash

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml
```

Wait ~1 minute, then check:

bash

```bash
kubectl get nodes
```

Should show `Ready`.

#### 9. Untaint the control plane

bash

```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

#### 10. Deploy a test pod

bash

```bash
kubectl run test-nginx --image=nginx
kubectl get pods -o wide
sudo crictl config runtime-endpoint unix:///run/containerd/containerd.sock
sudo crictl ps
```


### Step 1: Disable swap

bash

```bash
sudo swapoff -a
```

**What it does:** Immediately turns off all swap space on the system (the `-a` means "all swap devices/partitions listed in `/proc/swaps`").  
**Why needed:** kubelet checks at startup whether swap is active, and by default refuses to run at all if it is.  
**If you skip it:** kubelet crash-loops on startup with an error like `running with swap on is not supported`. Nothing else in the setup will work until this is off.

bash

```bash
sudo sed -i '/swap/s/^/#/' /etc/fstab
```

**What it does:** `/etc/fstab` is the file listing what gets mounted at boot. This command finds any line containing the word "swap" and puts a `#` at the start of it, commenting it out.  
**Why needed:** `swapoff -a` only lasts until your next reboot — at boot, `/etc/fstab` would remount swap automatically, undoing step 1 silently.  
**If you skip it:** Everything works fine right now, but the next time you reboot your PC, swap comes back on and kubelet will refuse to start again with no obvious explanation, since you'd have forgotten you needed to redo this.

### Step 2: Load kernel modules


```bash
cat << EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

**What it does:** Creates a config file listing two kernel module names, one per line. `tee` writes the piped input to that file (and also prints it to your terminal) — used here instead of `>` because `sudo` needs to apply to the _write_, not just reading the heredoc.  
**Why needed:** Files under `/etc/modules-load.d/` tell the system "load these kernel modules automatically on every boot." Without this file, you'd have to manually `modprobe` them again every time you restart your PC.  
**If you skip it:** The very next commands (`modprobe overlay`/`modprobe br_netfilter`) still work fine _this session_, but after a reboot, both modules are gone again and containerd/networking silently misbehaves until you remember to reload them.

bash

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

**What they do:** Actually loads each named kernel module right now, into the currently running kernel (the config file above only handles _future_ boots, not the present moment).  
**Why needed — `overlay`:** This is the filesystem type containerd uses to stack container image layers on top of each other efficiently (exact same concept as Docker's overlay2 storage driver, which you already know).  
**If you skip `overlay`:** containerd fails to start, or fails to pull/run any container, since it has no way to construct a container's filesystem.  
**Why needed — `br_netfilter`:** Lets the kernel's firewall rules (iptables) actually inspect traffic crossing a network bridge (the virtual switch pods' virtual network interfaces attach to).  
**If you skip `br_netfilter`:** Pods can start, but pod-to-pod networking breaks unpredictably — traffic between pods on the same node may not respect NetworkPolicy rules, or in some CNI setups, may not route correctly at all.

### Step 3: Sysctl networking params

bash

```bash
cat << EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```

**What it does:** Writes three kernel tuning parameters into a config file that `sysctl` reads.

- `net.bridge.bridge-nf-call-iptables = 1` — tells the kernel "when traffic crosses a bridge, also run it through iptables" (this is the actual _rule_ that `br_netfilter` from step 2 makes possible — step 2 loads the capability, step 3 turns it on).
- `net.bridge.bridge-nf-call-ip6tables = 1` — same thing, for IPv6 traffic.
- `net.ipv4.ip_forward = 1` — tells the kernel "you're allowed to forward packets between different network interfaces," i.e. act like a mini router. Pods on different virtual networks need your machine to forward packets between them.  
    **Why needed:** Without this, your earlier `br_netfilter` module load does nothing — the module being loaded and the setting being _enabled_ are two separate steps.  
    **If you skip it:** Pod-to-pod networking breaks in confusing, inconsistent ways — some pods might reach each other, some won't, NetworkPolicy enforcement silently doesn't work. Very annoying to debug because nothing gives you a clear error; things just don't connect.

bash

```bash
sudo sysctl --system
```

**What it does:** Reloads _all_ sysctl config files (including the one you just wrote) and applies them to the running kernel immediately.  
**Why needed:** Just writing the file in the step above doesn't apply it — Linux only reads these files at boot unless you tell it to re-read them now.  
**If you skip it:** The settings will only take effect after your next reboot — everything looks configured correctly on paper but isn't actually active yet, which is a common "why doesn't this work" trap.

### Step 4: containerd

bash

```bash
sudo pacman -S containerd
```

**What it does:** Installs the containerd package from Arch's official repos.  
**Why needed:** This is the program that actually runs your containers — pulls images, creates/starts/stops them. Nothing runs without it.  
**If you skip it:** Nothing else works — kubelet has nothing to talk to when it tries to start pods.

bash

```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

**What they do:** `containerd config default` prints containerd's built-in default configuration to your terminal. Piped into `tee`, that output gets written into `/etc/containerd/config.toml`. `mkdir -p` just makes sure that directory exists first (harmless if it already does).  
**Why needed:** containerd technically works with zero config file (it has built-in defaults), but you need an actual file on disk to _edit_ — specifically to fix the cgroup driver mismatch in the next command.  
**If you skip it:** containerd would start fine with defaults, but you'd have no file to change `SystemdCgroup` in, and that mismatch (explained below) will break kubelet.

**Manual edit** — changing `SystemdCgroup = false` to `SystemdCgroup = true` inside the file:  
**What it does:** Tells containerd to track each container's CPU/memory usage using `systemd`'s cgroup management, instead of managing cgroups directly itself ("cgroupfs" method).  
**Why needed:** kubelet (installed in step 5) _also_ has a cgroup driver setting, and it defaults to `systemd` on modern setups. If containerd and kubelet use different cgroup drivers, they each think they're tracking resources correctly but are stepping on each other.  
**If you skip it:** Pods might appear to start but resource limits get enforced incorrectly, or in many cases kubelet fails to even register the node correctly, and you get very confusing, hard-to-Google errors about cgroups mismatches.

bash

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

**What they do:** `restart` makes containerd reload with your edited config file (it doesn't watch the file live). `enable` tells systemd to start containerd automatically on every future boot.  
**Why needed `restart`:** Config file edits don't apply to an already-running process.  
**If you skip `restart`:** containerd keeps running with the _old_ config (SystemdCgroup still false), so your edit was pointless until you eventually restart it — same cgroup mismatch problem as above.  
**If you skip `enable`:** Everything works today, but after your next reboot, containerd doesn't start automatically, and kubelet crash-loops because it can't reach it — you'd have to know to manually start containerd every boot.

### Step 5: kubeadm, kubelet, kubectl

bash

```bash
yay -S kubeadm-bin kubelet-bin kubectl-bin
```

**What it does:** Installs three separate tools from the AUR (Arch's official repos don't package these, so a community-maintained AUR helper like `yay` builds/installs them).  
**Why needed:** These aren't optional — `kubelet` is the always-running agent, `kubeadm` bootstraps the cluster, `kubectl` is how you issue commands. All three are covered above in the previous explanation if you want the plain-English recap.  
**If you skip it:** Nothing to run at all — this is the actual Kubernetes software.

bash

```bash
sudo systemctl enable kubelet
```

**What it does:** Tells systemd to start kubelet automatically on boot (mirrors what you did for containerd).  
**Why needed:** Otherwise you'd have to manually start kubelet by hand every single boot before your cluster comes back up.  
**If you skip it:** Right now, nothing breaks — but after a reboot, kubelet won't be running, and your cluster looks completely dead until you remember to start it manually.  
**Note:** it _will_ crash-loop right after this command, printing errors — that's expected, because at this point there's no cluster config yet for it to use. Step 6 fixes that.

### Step 6: Initialize the control plane

bash

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

**What it does:** Bootstraps the entire cluster brain in one shot — generates certificates, starts etcd/API-server/scheduler/controller-manager as containers, and configures kubelet to manage them.  
**Why the `--pod-network-cidr` flag specifically:** This pre-declares the IP address range that pods (not nodes — pods) will get assigned from. `10.244.0.0/16` is a private range picked because it's the default that Calico (your CNI plugin from step 8) expects. If these two don't match, pods get IPs the CNI plugin never expected and networking breaks.  
**If you skip the flag entirely:** kubeadm still initializes, but with no pod CIDR declared, most CNI plugins fail to configure networking correctly, since they don't know what range to hand out — pods often get stuck in `ContainerCreating` forever.  
**If you skip this command entirely:** There's no cluster. Nothing after this point has anything to connect to.

### Step 7: Configure kubectl for your user

bash

```bash
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

**What they do:** `kubeadm init` generates a credentials file (`admin.conf`) that proves who's allowed to talk to the API server. `mkdir -p` makes the folder kubectl expects config in. `cp` copies the credentials there. `chown` changes the file's owner from `root` (since it was copied with `sudo`) to your regular user, using `$(id -u):$(id -g)` to grab your current user/group IDs automatically.  
**Why needed:** kubectl, by default, looks for exactly this file at exactly this path (`~/.kube/config`) to know both _where_ the cluster is and _who you are_ to it.  
**If you skip `chown`:** The file is owned by root, so kubectl (running as your normal user) can't read it, and every kubectl command fails with a permissions error — you'd have to prefix every single command with `sudo`, which is more annoying than doing this once.  
**If you skip all three:** kubectl has no idea a cluster exists or how to authenticate to it — every command just errors out saying it can't connect.

### Step 8: Install CNI (Calico)

bash

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml
```

**What it does:** Downloads a YAML file describing Calico's own components (as pods, ironically — Calico runs _inside_ the cluster it's setting up networking for) and tells the API server to create everything described in it.  
**Why needed:** Right after step 6, your node exists but has zero pod networking — no IPs get assigned, pods can't reach each other. This is the piece that actually implements the "virtual network" concept.  
**If you skip it:** Your node stays stuck showing `NotReady` forever in `kubectl get nodes`, and any pod you try to run gets stuck `Pending` or `ContainerCreating` indefinitely, since kubelet is waiting on networking to be ready before declaring the node healthy.

### Step 9: Untaint the control plane

bash

```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

**What it does:** Removes ("taints... minus" — the trailing `-` means _remove_ this taint, not add it) a label that marks this node as "control-plane only, don't schedule regular workloads here."  
**Why needed:** `kubeadm init` automatically applies this taint to protect the control plane's resources on real multi-node clusters. Since you only have one node total, leaving this taint on means there is literally no machine left for your own pods to run on.  
**If you skip it:** Every pod you try to deploy stays stuck in `Pending` forever, with `kubectl describe pod <name>` showing an error about no nodes tolerating the required taints.

### Step 10: Deploy a test pod

bash

```bash
kubectl run test-nginx --image=nginx
```

**What it does:** Asks the API server to create one pod running the `nginx` image.  
**Why needed:** This is your actual proof that everything above worked — scheduling, networking, and container execution all have to succeed for this to go green.  
**If any earlier step was broken:** This is usually the command where you'll _see_ it — the pod will sit in `Pending`, `ContainerCreating`, or `CrashLoopBackOff`, and each of those failure states points to a different earlier step being wrong.

bash

```bash
kubectl get pods -o wide
```

**What it does:** Lists pods with extra columns (`-o wide` adds node name, pod IP, etc.) so you can see _where_ it landed and _what IP_ it got.  
**Why needed:** Confirms the pod is `Running`, has an IP from the `10.244.0.0/16` range you declared in step 6, and is on your one node.

bash

```bash
sudo crictl config runtime-endpoint unix:///run/containerd/containerd.sock
sudo crictl ps
```

**What they do:** The first sets a default socket path so `crictl` doesn't warn you every time. `crictl ps` then lists running containers, talking _directly_ to containerd — bypassing kubelet and the API server entirely.  
**Why useful:** This is your closest equivalent to `docker ps` — a way to check "what does the actual container runtime think is running," independent of whether Kubernetes' own layers (API server, kubelet) are reporting things correctly. Handy for figuring out if a problem is in Kubernetes' logic or in containerd itself.  
**If you skip it:** Nothing breaks — this step is purely diagnostic, not required for the cluster to function.