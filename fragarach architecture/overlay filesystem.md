![[Pasted image 20260719034421.png]]

An overlay filesystem is made up of:

1. Upper layer — A Read-write layer.
2. Lower layer — A Read-only layer.
3. Work directory — A scratchpad for the filesystem to perform filesystem operations.
4. Merged — A logical merged view of upper, lower and work directories.

![[Pasted image 20260719034906.png]]

- **Lower (`lowerdir`):** The bottom layer. It is strictly **Read-Only**. The files here will never get modified.
    
- **Upper (`upperdir`):** The middle layer. It is **Read-Write**. Every single new file you create, change you save, or file you delete is recorded right here. 
    
- **Work (`workdir`):** An empty scratch space required by the Linux kernel. It acts as a staging area to prepare file operations atomically before they land in the Upper directory. You never interact with this folder directly.
    
- **Overlay/Merged (`mergeddir`):** The top view. This is the final unified perspective. When you navigate into this folder, you see the combined contents of both Lower and Upper simultaneously.

The flow: you will see all files in lowerdir initially when you open merged. When you modify any files, they will get modified for sure, but the changes are never made to the lower directory. Instead the changes are made in upper directory. 

```bash
sudo mount -t overlay overlay -o lowerdir=$HOME/overlay_demo/lower,upperdir=$HOME/overlay_demo/upper,workdir=$HOME/overlay_demo/work $HOME/overlay_demo/merged
```

To unmount:

```bash
sudo umount ~/overlay_demo/merged
```

https://medium.com/@aditya.raamesh/an-introduction-to-overlay-filesystem-205141fc5547