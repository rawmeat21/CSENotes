## Process Ownership: Real, Effective, and Saved UID/GID

Every process doesn't just have _one_ "owner" — it actually carries around multiple identity numbers at once. This seems weird at first, but it exists to solve a real security problem. Let's build it up with an example.

### The problem this solves

Imagine you're a regular user, `rawmeat`, and you want to change your own password. The command for that is `passwd`. But here's the catch: passwords are stored in `/etc/shadow`, a file that only **root** can write to. If `passwd` just ran with your normal permissions, what would happen? `/etc/shadow` can be modified by root only, so you will get error. 

So how does a regular user ever change their own password? This is exactly what the three UIDs are for.

### The three UIDs

#### 1. Real UID (RUID)

This is "who actually launched this process" — **your true identity**. It's used mainly for accounting/bookkeeping purposes (like figuring out who to blame in logs :). This is largely vestigial now — it doesn't do much active permission work.

**In our example:** Real UID = `rawmeat` (1000). That's genuinely you, running the command.

#### 2. Effective UID (EUID)

This is the one that actually matters for **permission checks right now**. When the kernel decides "can this process open this file / send this signal / do this privileged thing," it checks the **effective** UID, not the real one.

**In our example:** When you run `passwd`, the effective UID temporarily becomes `root` (0), even though _you_ (real UID 1000) launched it. This is done through something called the **setuid bit** — a special permission flag on the `passwd` binary that says "whoever runs this, temporarily run it as the file's owner (root)."

You can actually see this yourself:

bash

```bash
ls -l /usr/bin/passwd
```

```
-rwsr-xr-x 1 root root 68208 ... /usr/bin/passwd
```

See that `s` where you'd normally expect `x` in the owner's permissions? That's the setuid bit. It's what makes this whole trick work.

So while `passwd` runs:

- Real UID = 1000 (rawmeat — who really launched it)
- Effective UID = 0 (root — what permissions are actually checked)

This is how a normal user is able to write to a root-only file, but _only_ in this one controlled, narrow way (the `passwd` program only lets you edit your own password entry, nothing else).

#### 3. Saved UID (SUID — not to be confused with the setuid bit, unfortunately same abbreviation in casual speech)

This is a "parking spot." When a privileged process wants to **temporarily give up** its elevated permissions (drop from root down to a normal user) and then **come back to being privileged again later**, it needs somewhere to stash the privileged ID so it can reclaim it.

Think of it like a hotel keycard you hand to the front desk temporarily — you're not carrying it around (so you can't accidentally misuse it), but you can get it back on request without going through the whole check-in process again.

**Example:** A network daemon starts as root (needs root to bind to a low-numbered port like port 80). Once it's done that privileged setup step, it wants to drop down to a low-privilege user for safety (in case it gets hacked, the damage is limited). But it might occasionally need root privileges again for specific tasks (like writing to a log file only root can access). So:

- It sets effective UID → low-privilege user (day-to-day operation happens here, safely)
- Saved UID → still holds root, "parked," available to reclaim
- When needed, it can temporarily swap effective UID back to the saved (root) value, do the privileged task, then drop back down again

### GIDs work identically

Everything above applies the same way to **group** IDs — real GID, effective GID, saved GID — just for group-based permission checks instead of user-based ones. Same three-slot mechanism, same reasoning.

### Filesystem UID (Linux-specific)

It's used _only_ for file-access-permission checks specifically, and it's normally identical to the effective UID. It exists mainly for NFS (Network File System) edge cases.

### Quick summary table

|ID|Purpose|Normally equals|Changes when?|
|---|---|---|---|
|Real UID|Accounting — "who really started this"|Effective UID|Rarely|
|Effective UID|**Actual permission checks happen here**|Real UID|setuid programs, privilege drop/reclaim|
|Saved UID|"Parking spot" to reclaim dropped privileges|Effective UID (at process start)|When a program deliberately toggles privilege|

The core intuition to walk away with: **real UID = who you are, effective UID = who you're acting as right now, saved UID = who you can switch back to.**