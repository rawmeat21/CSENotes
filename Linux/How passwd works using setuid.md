## How `passwd` Works

### Are `setuid`/`setgid` "commands"? No, they're file permission bits

This is the most important thing to fix ito know. `setuid` and `setgid` are **not commands you run**. They're **permission flags stored in a file's metadata** on disk, the same metadata that stores read/write/execute permissions.

Every executable file already has permission bits like `rwxr-xr-x`. `setuid` and `setgid` are two _extra_ bits that can be layered on top of that. You set them using `chmod`, the same tool you'd use for any permission change:

bash

```bash
chmod u+s /usr/bin/passwd     # sets the setuid bit
chmod g+s somefile            # sets the setgid bit
```

You can see them in a directory listing:

bash

```bash
ls -l /usr/bin/passwd
```

```
-rwsr-xr-x 1 root root 68208 ... /usr/bin/passwd
```

Look closely at that `s` in `rws`. Normally that position would hold `x` (execute permission for the owner). The lowercase `s` means: _"execute permission is set, AND the setuid bit is also set."_ (If execute weren't set, you'd see a capital `S` instead — meaning setuid is on, but the file can't actually be executed, which is a broken/meaningless combination.)

So: **setuid is a property of the file `passwd` sitting on disk**, not an action you invoke each time. It's baked in once by whoever installed the system (or by root running `chmod`), and it stays there permanently until someone removes it.

### What that bit _causes the kernel to do_, at the moment you run it

Here's the actual mechanism, step by step, when you type `passwd`:

**1. You run the command as yourself.**

bash

```bash
$ passwd
```

At this instant, your shell asks the kernel to create (`fork`) and run (`exec`) the `passwd` binary. Your shell process has UID 1000 (you, `rawmeat`).

**2. The kernel checks the file's permission bits before running it.**  
This is the crucial moment. The kernel notices: _"this file has the setuid bit set, and is owned by root."_ Because of that, the kernel does something special when creating the new process: instead of giving the new `passwd` process an effective UID of 1000 (matching you, the person who ran it), it sets the new process's **effective UID to 0 (root)** — the UID of whoever _owns the file_, not whoever _ran_ it.

This is entirely automatic kernel behavior triggered by that one bit. Nothing in the `passwd` program's own code has to ask for this — it happens before the program's code even starts executing.

**3. Recall the three-UID system from earlier — this is exactly where it applies:**

- Real UID = 1000 (`rawmeat` —  who launched it)
- Effective UID = 0 (root — because of the setuid bit)
- Saved UID = 0 (also stashed, in case the program wants to toggle back and forth later)

**4. Now `passwd`'s own code runs, and it checks: "who really invoked me?"**  
It does this by looking at the **real UID** (1000 = you), _not_ the effective UID. This is important: the program is smart enough to distinguish "who has root permissions right now" (effective UID) from "who actually asked for this" (real UID). So: _"passwd checks to see who's running it and customizes its behavior accordingly."_

Since your real UID is 1000, `passwd` knows: _"okay, this is a normal user request — they should only be allowed to change their own password entry, nothing else."_ If real UID had been 0 (i.e., root itself ran `passwd`), it would allow changing _any_ user's password, because root is allowed to do that.

**5. `passwd` prompts you for your current password, verifies it, then asks for and sets the new one.**  
To actually write the new password hash, it needs to modify `/etc/shadow` — a file only root (UID 0) can write to. Because its **effective UID is 0** at this moment (thanks to the setuid bit from step 2), the kernel's permission check on that file write succeeds. If `passwd` didn't have the setuid bit, this write would fail with "Permission denied," exactly like it would if you tried `echo hi >> /etc/shadow` directly as yourself.

**6. `passwd` finishes and exits.** The process terminates entirely — there's no lingering root process hanging around. Your shell (still UID 1000 the whole time — the shell itself was never elevated, only the `passwd` process it launched) returns to a normal prompt.

### Putting it in one diagram

```
You (UID 1000) type: passwd
        │
        ▼
kernel sees passwd binary has setuid bit + owned by root
        │
        ▼
kernel creates passwd process with:
    real UID = 1000 (you)
    effective UID = 0 (root)   ← because file is setuid + root-owned
        │
        ▼
passwd's code checks REAL UID → sees 1000 → "only let them touch their own entry"
        │
        ▼
passwd prompts for old/new password, verifies old one
        │
        ▼
passwd writes new hash to /etc/shadow
    → this write succeeds because EFFECTIVE UID is 0 (root) right now
        │
        ▼
passwd process exits — elevated privilege disappears with it
        │
        ▼
back to your normal shell, still UID 1000, unaffected
```
